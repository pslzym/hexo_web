---
title: PCIE总线学习
tags: |-

  - Linux操作运维
permalink: pciezong-xian-xue-xi
id: 19
updated: '2016-09-22 14:47:14'
date: 2016-09-22 14:31:05
---



![](/uploads/2016/09/1.png)

PCIE总线的处理系统


PCI总线中有三类设备：
	PCI主设备、PCI从设备、桥设备
PCI主设备--->PCI Agent   网卡、显卡、声卡
PCI桥可以连接两条PCI总线，上游和下游，这两个PCI总线属于同一个总线域


PCI总线的负载：
	PCI总线的负载与总线的频率相关，其总线频率越高，能挂的负载越少。


![](/uploads/2016/09/2.png)

PCIE总线频率、带宽与负载均衡之间的关系

PCI总线传送方式：Posted和Non-Posted方式

PCI设备读写主存储器 ：DMA


![](/uploads/2016/09/3.png)

PCI设备读写主存储器 ：DMA

![](/uploads/2016/09/4.png)

存储器预与PCI总线域的划分


