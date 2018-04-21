---
title: Hive 2.1.0安装
tags: |-

  - Hadoop
  - Hive
permalink: hive2-1-0
id: 14
updated: '2016-09-19 23:24:22'
date: 2016-09-19 23:16:38
---

##HIVE环境搭建
第一次搭建HIVE环境也是坑，由于之前没有过mysql的相关运维和开发经验，导致浪费了不少时间。

###搭建Hadoop
Hadoop环境一定是要的，不管你是伪分布式模式还是全分布式模式，还是HA搭建，这个必要要有。

###HIVE安装
* 下载HIVE软件包
* Mysql安装和配置
* HIVE相关配置

####下载HIVE软件包
* 我下载的2.1.0版本的：[下载地址](http://apache.fayea.com/hive/)
* 由于2.1.0比较新，并且2.X的安装方法和1.X略有不同，相关资料也比较少，初次安装比较麻烦。

1.将 apache-hive-2.1.0-bin.tar.gz拷贝到/home/hadoop/Hive
2.解压tar -zxvf apache-hive-2.1.0-bin.tar.gz 
3.cd /home/hadoop/Hive/apache-hive-2.1.0-bin/conf
4.mv hive-default.xml.template hive-site.xml
5.mv hive-env.sh.template hive-env.sh
`Hive的主要配置文件就是hive-site.xml 和 hive-env.sh`

####安装mysql
1.	安装MySQL   （5.5.52）
```
sudo apt-get install mysql-server
sudo apt-get install mysql-client
```
2.	启动MySQL
	`/etc/init.d/mysql restart`
3.	登陆mysql做一些配置
```
mysql -uroot -p
Grant all on *.* to root@’%’ identified by ‘111111’ 设置登陆相关的权限，准许相关的用户远程登陆
create database hive;此库为Hive所使用的库
\q； 退出
```
4.	配置MySQL的配置文件
```
修改/etc/mysql/my.cnf
注释掉  bind-address           = 127.0.0.1
如果是其他版本，把禁止远程访问的配置项去掉就行了
重启MySQL	
如果Hive 启动报出相关mysql链接问题，可以现在另外一台机器上面验证其是否可以远程链接MySQL。
```
####修改Hive配置，启动Hive
1.修改hive-env.sh
```
HADOOP_HOME=/usr/local/hadoop
export HIVE_CONF_DIR=/home/hadoop/Hive/apache-hive-2.1.0-bin/conf
```
2.修改hive.
```

         <property>
		    <name>hive.exec.scratchdir</name>
		    <value>/tmp/hive</value>
		    <description>HDFS root scratch dir for Hive jobs which gets created with write all (733) permission. For each connecting user, an HDFS scratch dir: ${hive.exec.scratchdir}/&lt;username&gt; is created, with ${hive.scratch.dir.permission}.</description>
		  </property>
		  <property>
		    <name>hive.exec.local.scratchdir</name>
		    <value>/tmp/hive/local</value>
		    <description>Local scratch space for Hive jobs</description>
		  </property>
		  <property>
		    <name>hive.downloaded.resources.dir</name>
		    <value>/tmp/hive/resources</value>
		    <description>Temporary local directory for added resources in the remote file system.</description>
		  </property>
		  
		  <property>
		    <name>javax.jdo.option.ConnectionURL</name>
		    <value>jdbc:mysql://master/hive</value>
		    <description>
		      JDBC connect string for a JDBC metastore.
		      To use SSL to encrypt/authenticate the connection, provide database-specific SSL flag in the connection URL.
		      For example, jdbc:postgresql://myhost/db?ssl=true for postgres database.
		    </description>
		  </property>
		  
		  <property>
		    <name>javax.jdo.option.ConnectionUserName</name>
		    <value>root</value>
		    <description>Username to use against metastore database</description>
		  </property>
		  
		  <property>
		    <name>javax.jdo.option.ConnectionPassword</name>
		    <value>111111</value>
		    <description>password to use against metastore database</description>
		  </property>
  
  此配置主要根据MySQL的数据库相关配置来设置。
  
  3.Hadoop生成/tmp/hive目录，并改变相关权限。
  hadoop fs -mkdir /tmp/hive
  hadoop fs -chmod 777 /tmp/hive
  
  4.启动Hive
  		1.	2.x以上的Hive必须先运行：schematool -dbType mysql -initSchema
  		2.	hive
 这样就部署好了Hive...
 
####其他
如果是网络链接不了，试一试关闭防火墙：
	iptables -F 
  
  
  
	









