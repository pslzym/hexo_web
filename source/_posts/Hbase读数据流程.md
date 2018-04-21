---
title: Hbase读数据流程
tags: |-

  - Hbase
permalink: hbasedu-shu-ju-liu-cheng
id: 26
updated: '2016-09-27 23:46:25'
date: 2016-09-27 23:12:50
---

##Hbase读数据

* Hbase读数据流程
*Hbase -ROOT- 和 .META.表结构

###Hbase读数据流程
![](/uploads/2016/09/hbase------.png)
Hbase读数据流程图

    1.从zk里面获取-ROOT-表所在regionServer的信息。
    2.访问-ROOT-表所在的RegionServer，找到所要查询表的记录的二级索引。
    3.根据-ROOT-表里面的记录，查询对应的.META.表的regionServer，获取表真实记录所在的regionServer信息。
    4.根据在.MATE.所获取的信息，查询表所在的regionServer，并获取相应的数据。

###Hbase -ROOT- 和 .MATE.表结构

![](/uploads/2016/09/root.png)
ROOT表结构

![](/uploads/2016/09/meta.png)
META表结构

两张表的结构类似，因为都是用于定位表的位置。仔细看看两张表的ROWKEY的定义，对表的位置定位基本上就清楚啦。


