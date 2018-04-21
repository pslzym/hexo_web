---
title: Linux基础操作总结
tags: |-

  - Linux操作运维
permalink: linuxji-chu-cao-zuo-zong-jie
id: 4
updated: '2016-11-30 21:02:50'
date: 2016-09-17 17:20:44
---

##Linux基础操作

###samba服务器搭建
	yum install samba
	yum install samba-common
	yum install samba-client
	smbpasswd -a shaolong.psl
	service smb restart
	安装相关软件，设置相关用户的密码

###VirtualBox设置共享文件夹
虚拟机为ubuntu，物理机为window。

	设置共享文件夹：
	"mount -t vboxsf BaiduShare /mnt/bdshare/"

###ipmitool相关操作

	开启ipmi服务：
	service ipmi start

	查看BMC lan口信息
	ipmitool lan print
	
	通过ipmitool命令点灯。此方法用于在机房定位问题服务器。
	ipmitool chassis identify

	服务器进入PXE启动方式
	ipmitool chassis bootdev pxe
	ipmitool raw 0 2 3 

	改变服务器引导方式 
	ipmitool -I lan -H 10.1.199.212 -U ADMIN -P ADMIN chassis bootdev pxe
	ipmitool -I lan -H 10.1.199.212 -U ADMIN -P ADMIN chassis bootdev disk
	ipmitool -I lan -H 10.1.199.212 -U ADMIN -P ADMIN chassis bootdev cdrom

	服务器电源管理
	ipmitool -I lan -H 10.1.199.212 -U ADMIN -P ADMIN chassis power off  
	ipmitool -I lan -H 10.1.199.212 -U ADMIN -P ADMIN chassis power reset mitool -I lan -H 10.1.199.212 -U ADMIN -P ADMIN chassis power on   
	ipmitool -I lan -H 10.1.199.212 -U ADMIN -P ADMIN chassis power status

	登录oob1.tbc 
	ipmitool -I lanplus -H 10.1.199.XXX -U ADMIN -P ADMIN power reset 
	ipmitool -I lanplus -H 10.1.199.XXX -U ADMIN -P ADMIN sol activate

	
###Linux相关运维必备命令

	查看服务器运行时间和相关平均负载
	uptime

	查看板载硬件相关信息
	dmidecode

	进入无盘后修改机器密码
	mount /dev/sda2 /mnt  挂载OS盘到临时目录 
	cd /mnt 			  进入临时目录
	chroot . 			  将当前目录修改为根目录
	passwd 				  修改当前os密码

	查看Linux开启的端口
	netstat -nplt

	查看完整进程命令
	ps -efwww

	进程状态控制
	ctrl + z   挂起前端的进程   此时进程被挂起 得不到cpu的执行
	fg +number 将挂起的进程恢复到前台
	jobs显示被挂起的进程
	bg + number 将挂起的进程 启动得到cpu的执行

	window和linux传输文件方式
	可以安装一个lrzsz
	sudo apt-get install lrzsz
	通过sz和rz传输文件

	磁盘做LV：
	aliflash   创建pv需要修改/etc/lvm/lvm.conf   
	types   ===>   cat /proc/devise
	创建pv   pvcreate 
	创建vg   vgcreate 

	后台运行程序多种方法
	https://www.ibm.com/developerworks/cn/linux/l-cn-nohup/
	nohup cmd &
	screen -S XXX

	查看Linux所安装的软件包
	rpm -qa

	修改root密码
	sudo passwd 

	添加用户
	http://blog.51yip.com/linux/1137.html
	useradd test
	passwd test
	usermod -d /home/test -G test2 test
	userdel test
	last查看登录成功的用户记录
	lastb查看登录失败的用户记录

	查看usb相关设备
	lsusb
	
	lsof：
	http://man.linuxde.net/lsof
	http://blog.csdn.net/wyzxg/article/details/4971843
	http://blog.csdn.net/kozazyh/article/details/5495532
	
	lspci查看pcie设备

	tr替换：
	http://man.linuxde.net/tr
	echo "HELLO WORLD" | tr 'A-Z' 'a-z'

	du显示文件大小
	du -h
	du -ah
	du -h --max-depth=1
	#-h:用K、M、G的人性化形式显示
	#-a:显示目录和文件
	查看文件夹大小：
		du -sh XXX

	查看占用CPU最大的进程
		top 1 i c 可以看到占用cpu最大的进程

	shell命令运行符号&;&&
		command1&command2&command3     三个命令同时执行 
		command1;command2;command3     不管前面命令执行成功没有，后面的命令继续执行 
		command1&&command2             只有前面命令执行成功，后面命令才继续执行
		

###查看Linux版本信息
	cat /etc/issue
	uname -a 
	cat /proc/version
	lsb_release -a
	cat /etc/redhat-release
	应对不同的Linux发行版本，查看系统版本信息各不同，目前主要使用cat /etc/redhat-release

###常用Linux命令
####xargs
	
	http://man.linuxde.net/xargs	
	xargs命令是给其他命令传递参数的一个过滤器，也是组合多个命令的一个工具。它擅长将标准输入数据转
	换成命令行参数，xargs能够处理管道或者stdin并将其转换成特定命令的命令参数。xargs也可以将单行
	或多行文本输入转换为其他格式，例如多行变单行，单行变多行。xargs的默认命令是echo，空格是默认定
	界符。这意味着通过管道传递给xargs的输入将会包含换行和空白，不过通过xargs的处理，换行和空白将
	被空格取代。xargs是构建单行命令的重要组件之一.

	常用命令：
	ps -ef | grep XXX | awk '{print $2}' | xargs kill -9 

####grep
	http://man.linuxde.net/grep
	grep（global search regular expression(RE) and print out the line，全面搜索正则表达式
	并把行打印出来）是一种强大的文本搜索工具，它能使用正则表达式搜索文本，并把匹配的行打印出来。
	在多级目录中对文本进行递归搜索：
	grep "text" . -r -n
	grep  -rn "add_task_manual_submit" *

####find
	http://man.linuxde.net/find
	find命令用来在指定目录下查找文件。任何位于参数之前的字符串都将被视为欲查找的目录名。如果使用该命令时，不设置任何参数，则find命令将在当前目录下查找子目录与文件。并且将查找到的子目录和文件全部进行显示。
	find /home -name "*.txt"

####pgm
	安装：
		yum install tops-pgm
	不行的话最后面加*
		yum install taobao-pgm

	pgm主要是用于批量分发命令。对批量操作机器很有帮助。
	拷贝文件：
		pgmscp  -A -f ip.list -l root test.sh /home/
	批处理文件：
		pgm -b -p 10 -f ip.list -l root -A 'pwd'	

####ps 
	ps 命令：
	ps工具标识进程的5种状态码: 
	D 不可中断 uninterruptible sleep (usually IO) 
	R 运行 runnable (on run queue) 
	S 中断 sleeping 
	T 停止 traced or stopped 
	Z 僵死 a defunct (”zombie”) process 
	
	a  显示所有进程
	-a 显示同一终端下的所有程序
	-A 显示所有进程
	c  显示进程的真实名称
	-N 反向选择
	-e 等于“-A”
	e  显示环境变量
	f  显示程序间的关系
	-H 显示树状结构
	r  显示当前终端的进程
	T  显示当前终端的所有程序
	u  指定用户的所有进程
	-au 显示较详细的资讯
	-aux 显示所有包含其他使用者的行程 
	-C<命令> 列出指定命令的状况
	--lines<行数> 每页显示的行数
	--width<字符数> 每页显示的字符数
	--help 显示帮助信息
	--version 显示版本显示
	
	ps -A
	ps -u root
	ps -ef
	ps -l 将目前属于您自己这次登入的 PID 与相关信息列示出来
	ps aux 所有的正在内存中的程序

###ulimit 
    用来设置系统的资源限度。比如用来调节进程拥有的文件描述符数量
    http://man.linuxde.net/ulimit
    https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/

###防火墙
	SElinux操作：
		/usr/sbin/sestatus -v
		setenforce 0    #临时关闭SElinux,临时打开SElinux状态命令：setenforce 1
		vim /etc/selinux/config,然后将SELINUX改成disabled，这样即使重启操作系统SElinux也是关闭状态。
	iptables：
		service iptables status
		getenforce
		#service iptables stop   #临时关闭防火墙
   		#chkconfig iptables off    #永久关闭防火墙，开机不自启动，想自启动改成on


###服务器硬件基础知识
	cpu互联通过QPI总线   能保证cache的一致性
	cpu与PCH互联DMI总线
	CPU和高速设备连接一般是pcie
	SATA和sas协议
	cpu和内存连接是通过ddr4


###系统日志
![](/uploads/2016/09/OS----.png)

Linux日志输出文件和相关含义

![](/uploads/2016/09/OS-----1.png)

Linux日志级别
