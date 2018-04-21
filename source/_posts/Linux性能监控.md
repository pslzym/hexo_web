---
title: Linux性能监控
permalink: linuxxing-neng-jian-kong
id: 12
updated: '2016-09-19 11:50:40'
date: 2016-09-19 11:49:53
tags:
---

##Linux常用的性能监控
>* 基本的性能监控工具
>* 基本的调试工具

###基本的性能监控工具

	ifstat  网络监控。
	dstat： 查看系统初步的负载情况。
	lsof: 查看打开的文件。
	sar : sar 1 3000   用于看cpu整体的利用率  io的延迟等等
	mpstat   可以查看到每一个cpu内核的运行状态。排查网卡绑定的情况可以使用。
	mpstat -P ALL 1 3000

http://www.ibm.com/developerworks/cn/aix/library/au-lsof.html

http://linuxtools-rst.readthedocs.io/zh_CN/latest/tool/iostat.html

###基本的调试工具

	进程调试命令:truss、strace和ltrace
	查看系统函数调用的热度图：
	yum install perf
	perf top

###kingson讲课学习
	CPI   ：  一个instruction  需要多少个cpu来跑 ， 这个越小越好   最好是0.25一个clock最好的是跑4个instruction   很好的是0.5   一般是1
	
	1. increase CLK（and COST）
		increase CPU frequency
		increase cores
		increase sockets
		Hyper-Threading
	2.Reduce CPI
	3.Reduce path length
		SW(compiler,runtime config, programming)
		Better compilers
		Profile guided optimizations
	First thing to do?
	
	branch mispredict ratio  cpu猜测错误的频率0.01    好  0.05差
