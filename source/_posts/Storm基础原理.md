---
title: Storm基础原理
tags: |-

  - storm
permalink: stormji-chu-yuan-li
id: 24
updated: '2016-09-24 18:55:41'
date: 2016-09-24 18:51:54
---

##Storm原理图

* 主架构
* 主流程

###主架构

              主节点     工作节点          作业
    Storm     Nimbus     supervisor      topologies,死循环
    hadoop    Jobtracker  Tasktracker     MapReduce Job，执行完自动结束

![](/uploads/2016/09/strom1.png)
主要架构图

Nimbus 和supervisor之间所有的协调工作时通过一个zookeeper集群。
Nimbus 进程和supervisor进程是无法直接连接和无状态的；所有的状态维持在zookeeper中或保持在本地磁盘上



Nimbus 负责在集群分发代码，top只能在Nimbus机器上面提交，将任务分配给其他机器。
Supervisor监听分配给它的节点，根据Nimbus的委派在必要时启动和关闭工作进程。每个工作进程执行top的一个子集。一个运行中的top由很多运行在很多机器上的工作进程组成。
在Storm中有对于流stream的抽象。流是一个不间断的无界的连续的tuple，注意Storm在建模事件流时，把流中德事件抽象为tuple即元组

![](/uploads/2016/09/strom2.png)
工作图

###主流程
Storm认为每个stream都有一个源，也就是原始元组的源头，叫做Spout（管口）
处理stream内的tuple，抽象为Bolt，bolt可以消费任意数量的输入流，只要将流方向导向该bolt，同时它也可以发送新的流给其他bolt使用，这样一来，只要打开特定的spout再将spout中流出的tuple导向特定的bolt，又bolt对导入的流做处理后再导向其他bolt或者目的地。

可以认为spout就是水龙头，并且每个水龙头里流出的水是不同的，我们想拿到哪种水就拧开哪个水龙头，然后使用管道将水龙头的水导向到一个水处理器（bolt），水处理器处理后再使用管道导向另一个处理器或者存入容器中。


![](/uploads/2016/09/strom3.png)
spout和bolt工作图


这是一张有向无环图，Storm将这个图抽象为Topology(拓扑)，Topo就是storm的Job抽象概念，一个拓扑就是一个流转换图

图中每个节点是一个spout或者bolt，每个spout或者bolt发送元组到下一级组件，广播方式。

而Spout到单个Bolt有6种grouping方式，后续细讲。


Storm将流中元素抽象为tuple，一个tuple就是一个值列表value list，list中的每个value都有一个name，并且该value可以是任意可序列化的类型。拓扑的每个节点都要说明它所发射出的元组的字段的name，其他节点只需要订阅该name就可以接收处理。

![](/uploads/2016/09/strom4.png)

Streams：消息流
消息流是一个没有边界的tuple序列，而这些tuples会被以一种分布式的方式并行创建和处理。 每个tuple可以包含多列，字段类型可以是： integer, long, short, byte, string, double, float, boolean和byte array。 你还可以自定义类型 — 只要你实现对应的序列化器。


Spouts：消息源
Spouts是topology消息生产者。Spout从一个外部源(消息队列)读取数据向topology发出tuple。 消息源Spouts可以是可靠的也可以是不可靠的。一个可靠的消息源可以重新发射一个处理失败的tuple， 一个不可靠的消息源Spouts不会。

Spout类的方法nextTuple不断发射tuple到topology，storm在检测到一个tuple被整个topology成功处理的时候调用ack, 否则调用fail。
storm只对可靠的spout调用ack和fail。

Bolts：消息处理者
消息处理逻辑被封装在bolts里面，Bolts可以做很多事情： 过滤， 聚合， 查询数据库等。
Bolts可以简单的做消息流的传递。复杂的消息流处理往往需要很多步骤， 从而也就需要经过很多Bolts。第一级Bolt的输出可以作为下一级Bolt的输入。而Spout不能有一级。

Bolts的主要方法是execute（死循环）连续处理传入的tuple，成功处理完每一个tuple调用OutputCollector的ack方法，以通知storm这个tuple被处理完成了。当处理失败时，可以调fail方法通知Spout端可以重新发送该tuple。

流程是： Bolts处理一个输入tuple, 然后调用ack通知storm自己已经处理过这个tuple了。storm提供了一个IBasicBolt会自动调用ack。
Bolts使用OutputCollector来发射tuple到下一级Blot。



