---
title: Storm事务-批处理原理
tags: |-

  - storm
permalink: stormshi-wu-pi-chu-li-yuan-li
id: 35
updated: '2016-10-08 13:59:01'
date: 2016-10-08 13:54:36
---

##Storm事务-批处理原理

1.	事务
对于容错机制，Storm通过一个系统级别的组件acker，判断tuple是否发送成功，进而spout可以重发该tuple，保证一个tuple在出错的情况下至少被重发一次。
但是在需要精确统计tuple的场景下（销售金额），希望每个tuple“被且仅被处理一次”。storm引入了Transactional Topology。它可以保证每一个“tuple”被且仅被处理一次，这样就可以实现一种非常准确，且高度容错方式来实现计数类应用。

2.	批处理
	逐个处理单个tuple，增加很多开销，如写库、输出结果频率过高。事务处理单个tuple效率比较低，因此storm中引入batch处理。
	批处理是一次性处理一批（batch）tuple，事务可以确保该批次要么全部处理成功，如果有处理失败的则全部不计，Storm会对失败的批次重新发送，且确保每一个batch被且仅被处理一次。
	
###事务机制和原理
对于只处理一次的需要，从原理上来讲，需要在发送tuple的时候带上，txid，在需要事务处理的时候，根据该txid是否以前已经处理成功来决定是否进行处理，当然需要把txid和处理结果一起做保存。
在事务batch处理中，一批tuple赋予一个txid，为了提高batch之间处理的并行度，storm采用了pipeline处理模型，这样多个事务可以并行执行，但是commit的是严格按照顺序的。

Storm事务处理中，把一个batch的计算分成两个阶段processing和commit阶段：
processing阶段：多个batch可以并行计算。
commit阶段：batch之间强制按照顺序进行提交。
![](/uploads/2016/10/--1.png)

####事务Topologies
使用Transactional Topologies的时候，storm为你做下面这些事情。

* 管理状态：Storm把所有实现Transactional Topologies所必须的状态保存在zookeeper里面，包括当前transaction id 以及定义每一个batch的一些元数据。
* 协调事务：Storm帮你管理所有事情，如帮你决定在任何一个时间点是该processing还是该committing.
* 错误检测: Storm利用acking框架来高效地检测什么时候一个batch被成功处理了，被成功提交了，或者失败了。Storm然后会相应地replay对应的batch。你不需要自己手	动做任何acking或者anchoring (emit时发生的动作)。
* 内置的批处理API: Storm在普通bolt之上包装了一层API来提供对tuple的批处理支持。Storm管理所有的协调工作，包括决定什么时候一个bolt接收到一个特定transaction的所有tuple。Storm同时也会自动清理每个transaction所产生的中间数据。


事务性的spout需要实现ITransactionalSpout，这个接口包含两个内部接口类Coordinator和Emitter。在topology运行的时候，事务性的spout内部包含一个子Topology，结构图如下：

![](/uploads/2016/10/--2.png)

这里面有两种类型的tuple，一种是事务性的tuple，一种是batch中的tuple；
coordinator 开启一个事务准备发射一个batch时候，进入一个事务的processing阶段，会发射一个事务性 tuple(transactionAttempt & metadata)到”batch emit”流

Emitter以all grouping(广播)的方式订阅coordinator的”batch emit”流，负责为每个batch实际发射tuple。发送的tuple都必须以TransactionAttempt作为第一个field，storm根据这个field来判断tuple属于哪一个batch。

coordinator只有一个，emitter根据并行度可以有多个实例

coordinator只有一个，emitter根据并行度可以有多个实例


####TransactionAttempt 和 元数据

TransactionAttempt包含两个值：一个transaction id，一个attempt id。transaction id的作用就是我们上面介绍的对于每个batch中的tuple是唯一的，而且不管这个batch    replay多少次都是一样的。
attempt id是对于每个batch唯一的一个id， 但是对于同一个batch，它replay之后的attempt id跟replay之前就不一样了，
我们可以把attempt id理解成replay-times， storm利用这个id来区别一个batch发射的tuple的不同版本

metadata(元数据)中包含当前事务可以从哪个point进行重放数据，存放在zookeeper中的，spout可以通过Kryo从zookeeper中序列化和反序列化该元数据。

![](/uploads/2016/10/--3.png)
内部处理流程

####事务性Bolt
* BaseTransactionalBolt处理batch在一起的tuples，对于每一个tuple调用调用execute方 法，而在整个batch处理(processing)完成的时候调用finishBatch方法。如果BatchBolt被标记成Committer，则 只能在commit阶段调用finishBatch方法。一个batch的commit阶段由storm保证只在前一个batch成功提交之后才会执行。并且它会重试直到topology里面的所有bolt在commit完成提交。那么如何知道batch的processing完成了，也就是bolt是否接收处理了batch里面所有的tuple；在bolt内部，有一个 CoordinatedBolt的模型。
CoordinateBolt具体原理如下：

![](/uploads/2016/10/--4.png)

每个CoordinateBolt记录两个值：有哪些task给我发送了tuple（根据topology的grouping信息）；我要给哪些task发送信息（同样根据groping信息）。

等所有的tuple都发送完了之后，CoordinateBolt通过另外一个特殊的stream以emitDirect的方式告诉所有它发送过 tuple的task，它发送了多少tuple给这个task。下游task会将这个数字和自己已经接收到的tuple数量做对比，如果相等，则说明处理 完了所有的tuple。

下游CoordinateBolt会重复上面的步骤，通知其下游。
