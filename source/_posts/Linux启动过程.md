---
title: Linux启动过程
tags: |-

  - Linux操作运维
permalink: linuxqi-dong-guo-cheng
id: 9
updated: '2016-09-29 15:52:53'
date: 2016-09-18 14:21:45
---

##Linux 启动过程

![](/uploads/2016/09/1315187575_42333049--2-.png)


vmlinuxz是真正的内核文件，initramfs用来协助内核挂载文件系统。
/usr/src/kernels下面存放的是内核的源码和相关编译好的ko文件。

Linux 启动过程图

pxe启动过程：
	
	PXE Client 向 UDP 67端口 广播 DHCPDDISCOVER 消息.
	DHCP SERVER 或者 DHCP Proxy 收到广播消息后,发送DHCPOFFER(包含ip地址)消息 到 PXE Client的 68 端口.
	PXE Client 发送 DHCPREQUEST 消息到 DHCP SERVER ,获取启动文件(boot file name).
	DHCP SERVER 发送DHCPACK(包含Network Bootstrap Program file name)消息 到PXE Client.
	PXE Client 向 Boot Server 获取 NBP(Network Bootstrap Program) 文件.
	PXE Client 从 TFTP SERVER 下载 NBP,然后在客户端执行NBP文件

	此时已经获取到NBP文件：

	pxelinux.0(NBP)文件：获取安装文件
			RAMOS


    忘记root密码：
        grub页面进入   single
        修改完密码之后init 5

    系统出问题进入不了OS
    grub界面修改启动程序   init=/bin/bash
    此时系统只是挂载了跟目录，并且为只读。
    mount -o remount,rw /

 

相关细节：

http://www.cnblogs.com/wwang/archive/2010/10/27/1862222.html
http://haobo.blog.51cto.com/2893071/589479

http://blog.csdn.net/jx_jy/article/details/12783559

http://www.doc88.com/p-474114520477.html
