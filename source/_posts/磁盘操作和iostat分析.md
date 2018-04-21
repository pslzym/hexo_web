---
title: 磁盘操作和iostat分析
tags: |-

  - Linux操作运维
permalink: ci-pan-cao-zuo-he-iostatfen-xi
id: 11
updated: '2016-09-19 11:05:52'
date: 2016-09-19 10:49:50
---

##磁盘运维
	lsblk：
		显示服务器上面块设备的相关信息
	fdisk：
		对容量偏小的磁盘进行相关操作
	parted：
		对大容量的磁盘进行相关操作
		eg： parted /dev/dfc mklabel gpt mkpart 1 ext4 1 6400G
		http://zhangmingqian.blog.51cto.com/1497276/1068779
	smartctl:
		查看磁盘相关硬件信息
	hdparm:
		hdparm  和dd差不多，是用来压测磁盘的。可以用来查看硬盘信息。

	lsscsi:
		查看每个硬盘的型号和厂家  

	df -T -h  可以查看文件系统类型

	
	partprobe要内核更新分区表（对于磁盘分区之后，调用该命令更新内核分区表）

	tune2fs:
		查看分区的相关信息
		tune2fs -l /dev/sda5

	格式化磁盘：
		#!/bin/bash -
		for i in {a..n};
		do
			nohup &>/dev/null dd if=/dev/zero of=/dev/sd$i bs=1G count=4000 & 
		done


##性能监控
	iotop
		主要用于查看相关进程的磁盘访问情况。

	iostat
		iostat -xk 1

![](/uploads/2016/09/iostat.png)

无大量压力下，系统的io情况。

    |字段|含义|
    |-------|:------------------- |
    |rrqm/s	|(read request merge)/(读合并数  HDD)主要是合并IO，让落到磁盘上面的请求量更少|
    |wrqm/s |(write request merge)/(写合并数  HDD)|
    |r/s 	|read iops|
    |w/s	|write iops|
    |rkB/s	|read带宽|
    |wkB/s	|write带宽|
    |avgrq-sz|平均每次设备I/O操作的数据大小 (扇区)|
    |avgqu-sz|平均I/O队列长度.即 delta(aveq)/s/1000 (因为aveq的单位为毫秒)|
    |await	|平均每次设备I/O操作的等待时间 (毫秒).	|
    |r_await | 平均每次设备I/O读操作的等待时间 (毫秒).|
    |w_await | 平均每次设备I/O写操作的等待时间 (毫秒).|
    |svctm   | 平均每次设备I/O操作的服务时间 (毫秒).即 delta(use)/delta(rio+wio)|
    |util	|一秒中有百分之多少的时间用于 I/O 操作,或者说一秒中有多少时间 I/O 队列是非空的.即 delta(use)/s/1000 (因为use的单位为毫秒)|

一般ssd的await 在190左右

http://www.ha97.com/4546.html
http://www.php-oa.com/2009/02/03/iostat.html

![](/uploads/2016/09/dd1024.png)

##磁盘和Linux驱动关系
目前很多磁盘为了增大空间，会有一个逻辑扇区和物理扇区。

![](/uploads/2016/09/sector.png)

物理扇区是磁盘物理存储的最小单元，逻辑扇区是为了兼容Linux内核扇区的大小，现在Linux内核默认的扇区大小为512。逻辑扇区是OS访问磁盘的最小单元。iostat出现的avgrq-sz的单位就是逻辑扇区。

物理扇区和逻辑扇区会产生对齐的问题，在格式化磁盘的时候要注意。

http://noops.me/?p=747

##Inode和文件存储
	文件存储在硬盘上，硬盘的最小操作单元是扇区，每个扇区是512字节.
	操作系统操作磁盘的时候不会直接去操作扇区，这样效率很低。操作系统一般是操作block，每个block是4K.
	分区上的文件系统相关信息可以通过：tune2fs -l /dev/sda1 或者 dumpe2fs -h /dev/sda1来查看。
	文件的数据都存储在"块"中，相关的文件原信息存储在inode中。
	inode一般存储的是文件的名称、修改时间、权限、创建者、文件大小等等。
	
	文件的inode信息可以通过stat XXX查看到。
	inode的大小会消耗磁盘空间，所以磁盘格式化的时候，操作系统会自动将硬盘分为两个区域，一个是数据区，存放真实的数据；一个是inode区，存放inode 相关的信息。（df -i）

![](/uploads/2016/09/io-----.png)
io细致图
![](/uploads/2016/09/io--.png)
整体的逻辑架构图
![](/uploads/2016/09/--IO--.png)
IO经典架构图

http://way4ever.com/?p=359


