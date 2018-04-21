---
title: Storm集群搭建
tags: |-

  - storm
permalink: stormji-qun-da-jian
id: 22
updated: '2016-10-05 15:22:12'
date: 2016-09-24 16:19:57
---

##storm 集群搭建

* zookeeper环境
* Storm配置
* Storm运行

###zookeeper环境
zookeeper环境搭建参考zookeeper搭建相关博文

###Storm配置

    1.	下载软件包
        1. 在博主学习的时候，最新的storm版本是1.02。相关下载链接：http://storm.apache.org/releases/1.0.2/index.html
        2. 解压到相关目录
            tar -zxvf apache-storm-1.0.2.tar.gz
            博主比较懒，直接解压在hadoop的主目录中。/home/hadoop/storm/apache-storm-1.0.2
        3.配置相关文件
           storm最主要的配置文件是：/home/hadoop/storm/apache-storm-1.0.2/conf/storm.yaml
		相关配置如下：
		# Licensed to the Apache Software Foundation (ASF) under one

	 storm.zookeeper.servers:
	     - "master"
	     - "slave1"
	     - "slave2"
 	nimbus.host: "master"   nimbus的主机ip或者名称

 	nimbus.seeds: ["master", "slave1", "slave2", "slave3", "slave4"]  所有机器列表
 	
 	storm.local.dir: "/home/hadoop/storm/apache-storm-1.0.2/data" storm文件存放目录，需要提前建立好，并赋予相关权限
 	
 	supervisor.slots.ports:    相关slots端口号
 	 - 6700
      - 6701
      - 6702
      - 6703
  
     ui.port: 8081  ui界面的端口号
  
     注意每一个配置项开头必须留一个空格
  
 
###Storm 运行
 * 检测nimbus节点是否能正常运行
 * 拷贝相关文件，启动整个集群

####检测nimbus节点是否能正常运行
     1.进入storm的bin目录。
     2.运行./storm nimbus,如果能正常输出相关命令，说明相关软件环境已经配置完成。
     3.ctr + c 杀掉刚刚启动的进程
 
####拷贝相关文件，启动整个集群
    1.拷贝storm软件的文件夹到集群中的每一台机器。
    2.逐个启动相关软件：
 	nimbus ： nohup ./storm nimbus &
 	supervisor : nohup ./storm supervisor &
    3.检查集群启动状态：
        运行nohup ./storm ui &
	访问网址：master:8081查看相关集群的运行状态。
