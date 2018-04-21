---
title: Storm分组方式和并发
tags: |-

  - storm
permalink: stormfen-zu-fang-shi-he-bing-fa
id: 32
updated: '2016-10-07 00:18:38'
date: 2016-10-07 00:16:54
---

##Storm分组方式和并发
* 基础理论
* Storm分组方式
* Storm并发

###基础理论
* Nodes(服务器):指配置在一个Storm集群中的服务器，会执行top的一部分运算。一个Storm集群可以包括一个或者多个工作node。

* Worker(JVM虚拟机): 指一个node上相互独立运行的JVM进程。每一个node可以配置运行一个或者多个worker。一个top会分配到一个或者多个worker上运行。  

* Executor(线程):指一个worker的JVM进程中运行的Java线程。多个task可以指派给同一个executor来执行。除非是明确指定，Storm默认会给每一个executor分配一个task。

* Task(Spout/bolt实例):task是spout和bolt的实例，它们的nextTuple（）和execute（）方法会被executor线程调用执行。task代表最大并发度。executor数代表实际并发数。

###Storm并发
![](/uploads/2016/10/9DD2F949-EADB-41FD-BE40-CB1DBB59D2D4.png)
并发举例

Config conf = new Config();
//设置两个worker
conf.setNumWorkers(2); 
//spout数目线程数2
topologyBuilder.setSpout("blue-spout", new BlueSpout(),2); 
//线程数为2，并发度为4   见图
topologyBuilder.setBolt("green-bolt", new GreenBolt(), 2).setNumTasks(4).shuffleGrouping("blue-spout");
//线程数为6
topologyBuilder.setBolt("yellow-bolt", new YellowBolt(), 6).shuffleGrouping("green-bolt");
StormSubmitter.submitTopology(
        "mytopology",
        conf,
    topologyBuilder.createTopology()
    );


###Storm分组方式
数据流分组定义了一个数据流中的tuple如何发送给top中不同bolt的task。
注：不是一个spout或bolt emit到多个bolt（广播方式）
目前官方有八种

* Shuffle Grouping(随机分组)：这种方式会随机分发tuple给bolt的各个task，每个bolt实例接收到的相同数量的tuple。

* Field Grouping(按字段分组)：根据指定的字段进行分组。（重点）

* Non Grouping: 无分组， 这种分组和Shuffle grouping是一样的效果，多线程下不平均分配。

* All Grouping： 广播发送。The stream is replicated across all the bolt's tasks. Use this grouping with care.

* Global Grouping: 全局分组， 这个tuple被分配到storm中的一个bolt的其中一个task。再具体一点就是分配给id值最低的那个task。适合场景：想象不到。

* Direct Grouping: 直接分组， 这是一种比较特别的分组方法，用这种分组意味着消息的发送者决定由消息接收者的哪个task处理这个消息。 只有被声明为Direct Stream的消息流可以声明这种分组方法。而且这种消息tuple必须使用emitDirect方法来发射。消息处理者可以通过TopologyContext来或者处理它的消息的taskid (OutputCollector.emit方法也会返回taskid)

* Partial Key grouping: The stream is partitioned by the fields specified in the grouping, like the Fields grouping, but are load balanced between two downstream bolts, which provides better utilization of resources when the incoming data is skewed. This paper provides a good explanation of how it works and the advantages it provides.（最新新增）

* Local or shuffle grouping: If the target bolt has one or more tasks in the same worker process, tuples will be shuffled to just those in-process tasks. Otherwise, this acts like a normal shuffle grouping.


