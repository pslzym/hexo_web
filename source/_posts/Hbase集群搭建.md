---
title: Hbase集群搭建
tags: |-

  - Hbase
  - Hadoop
permalink: hbaseji-qun-da-jian
id: 23
updated: '2016-09-25 01:33:46'
date: 2016-09-24 16:40:24
---

##Hbase环境搭建
* 搭建zookeeper集群和HDFS集群
* 安装Hbase软件
* 启动集群
* shell客户端连接

###搭建zookeeper集群和HDFS集群
zookeeper 对于Hbase集群是必不可少的。zookeeper为hadoop集群提供协同服务。
HDFS为Hbase提供体层的存储服务
zookeeper和HDFS搭建参考相关博文

###安装Hbase软件
目前Hbase最新稳定版本为1.2.3。在官网相关网页下载相关软件包。

1.将软件包解压到相关路径
2. 进入配置文件目录，修改相关配置文件


    hbase-site.xml
    <configuration>
	<property>
                <name>hbase.rootdir</name>
                <value>hdfs://Master:9000/hbase</value>
        </property>
		<!-- 指定hbase是分布式的 -->
        <property>
                <name>hbase.cluster.distributed</name>
                <value>true</value>
        </property>
		<!-- 指定zk的地址，多个用“,”分割 -->
        <property>
                <name>hbase.zookeeper.quorum</name>
                <value>Master:2181,Slave1:2181,Slave2:2181</value>
        </property>
</configuration>

    hbase-env.sh
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
    export HBASE_OPTS="-XX:+UseConcMarkSweepGC"
    export HBASE_MASTER_OPTS="$HBASE_MASTER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
    export HBASE_REGIONSERVER_OPTS="$HBASE_REGIONSERVER_OPTS -XX:PermSize=128m -XX:MaxPermSize=128m"
    export HBASE_MANAGES_ZK=false

    regionservers
    Slave1
    Slave2
    Slave3
    Slave4

    以上三个文件主要配置zookeeper和HDFS相关配置信息

###启动集群
1.启动集群
进入软件包的bin目录，执行start-hbase.sh便可以启动Hbase集群。

2.web界面访问http://master:16010/master-status可以看到集群的状态

3.在HMaster上面有HMaster进程，在HRegion上面有一个HReginServer的进程。
4.启动HMaster的backup进程。
在其他机器上进入bin目录启动./hbase_daemon.sh start master

###shell客户端连接
进入软件包的bin目录，运行./hbase shell即可进入Hbase的shell client端。




