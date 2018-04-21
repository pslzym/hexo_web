---
title: Hadoop HA 部署方案
tags: |-

  - Hadoop
permalink: hadoop-ha-bu-shu-fang-an
id: 40
updated: '2016-10-09 17:20:42'
date: 2016-10-09 17:18:23
---

##Hadoop HA 部署方案
*	原理
* 	部署方法

###原理

![](/uploads/2016/10/HA.png)
原理图

```
HA的方案比单节点Master多了几个组件：
zookeeper集群：   主要是用来维护和选举Master
ZKFC:            每一个master上必须运行一个，主要是用来监控master的状态，为选举active的master做状态监控。
JN（JournalNode）集群：  主要是用来存储Master的原始数据，log和Matadata。防止log和matadata丢失。
``` 

###部署方法
官方连接：https://hadoop.apache.org/docs/r2.6.0/hadoop-project-dist/hadoop-hdfs/HDFSHighAvailabilityWithQJM.html#Configuration_details

主要配置文件:
hdfs-site.xml

```
<configuration>
        <property>
  		<name>dfs.nameservices</name>
  		<value>pslcluster</value>
	</property>
        <property>
    		<name>dfs.ha.namenodes.pslcluster</name>
  		<value>nn1,nn2</value>
	</property>
        <property>
  		<name>dfs.namenode.rpc-address.pslcluster.nn1</name>
  		<value>master:8020</value>
	</property>
	<property>
  		<name>dfs.namenode.rpc-address.pslcluster.nn2</name>
  		<value>slave4:8020</value>
	</property>
	<property>
  		<name>dfs.namenode.http-address.pslcluster.nn1</name>
  		<value>master:50070</value>
	</property>
	<property>
  		<name>dfs.namenode.http-address.pslcluster.nn2</name>
  		<value>slave4:50070</value>
	</property>
        <property>
  		<name>dfs.namenode.shared.edits.dir</name>
  		<value>qjournal://slave1:8485;slave2:8485;slave3:8485/mycluster</value>
	</property>
	<property>
  		<name>dfs.client.failover.proxy.provider.pslcluster</name>
  		<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
	<property>
  		<name>dfs.ha.fencing.methods</name>
  		<value>sshfence</value>
	</property>
	<property>
  		<name>dfs.ha.fencing.ssh.private-key-files</name>
  		<value>/home/hadoop/.ssh/id_rsa</value>
	</property>

	<property>
  		<name>dfs.journalnode.edits.dir</name>
  		<value>/usr/local/hadoopHA/journal/data</value>
	</property>
	<property>
   		<name>dfs.ha.automatic-failover.enabled</name>
   		<value>true</value>
 	</property>
</configuration>
```

core-site.xml

```
<configuration>
	<property>
 	 	<name>fs.defaultFS</name>
  		<value>hdfs://pslcluster</value>
	</property>
	<property>
   		<name>ha.zookeeper.quorum</name>
   		<value>master:2181,slave1:2181,slave2:2181</value>
 	</property>
	<property>
   		<name>hadoop.tmp.dir</name>
   		<value>/usr/local/hadoopHA/tmp</value>
 	</property>
</configuration>
```

slaves

```
Slave1
Slave2
Slave3
```

还必须配置一下hadoop-env.sh 中的java环境

启动过程：

1.	首先手动启动journalnode
	hadoop-daemon.sh start journalnode在相关的

2.	在其中一台namenode上面格式化
	namenode hdfs namenode -format
	
3.	启动刚刚格式化完成的namenode :
	启动一个namenode
	
4.	在另外一台master上面的copy相关原始数据
	hdfs namenode -bootstrapStandby
	
5.	停止所有服务
	./stop-all.sh
	
6.	初始化zkfc
	hdfs zkfc -formatZK
	
7.	启动集群
	start-dfs.sh
	
部署好之后可以看一下相关的进程和端口：
http://master:50070
尝试put一个数据到相关的集群。
	


