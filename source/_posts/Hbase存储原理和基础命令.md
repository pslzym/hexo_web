---
title: Hbase存储原理和基础命令
tags: |-

  - Hbase
  - Hadoop
permalink: hbasecun-chu-yuan-li-he-ji-chu-ming-ling
id: 25
updated: '2016-09-27 23:58:42'
date: 2016-09-25 01:48:27
---

##Hbase原理和基础命令
* Hbase存储基本原理
* Client端基础命令

###Hbase存储基本原理

![](/uploads/2016/09/Hbase1.png)
存储原理图

![](/uploads/2016/09/Hbase2-1.png)
大表存储图

数据管理机制
1、hbase中的表很大---bigtable，都是分布式存储在集群的各个regionserver上

2、分布式存储时，需要对表进行切分，首先是按行切分成若干个hregion

3、表的每一个hregion都会被一个regionserver所管理

4、每一个hregion随着插入数据的增多，一旦达到一个阈值，会被regionserver分裂成两个

5、在一个hregion内部还会被按照列族切分成若干个store单元

6、每一个store又会被切分成若干个store file(HFile)存储到hdfs文件系统中

7、每一个store对象中会维护一个内存缓存 memstore，用户查询数据时首先会去memstore中进行命中

8、客户端往表中插入数据，首先会在hlog日志中进行记录，然后定期进行flush合并


###Client端基础语法

![](/uploads/2016/09/Hbase3.png)

![](/uploads/2016/09/Hbase4.png)
Hbase基础表结构

1.创建表的基础命令
	create ‘user_info’, 'base_info', 'extra_info';
	指定版本号：
	create 'user_info', {NAME =>'base_info', VERSION => 3}, 'extra_info'
	
2.插入数据
	put 'user_info', '0001', 'base_info:phone', '13967109402'
	put 'user_info', '0001', 'base_info:name', 'angleababy'
	
3.全表扫描
	scan ‘user_info’ 
	
4.查看数据
	get 'user_info','0001', 'base_info:name', 'base_info:age'
	get 'user_info2', '0001', { COLUMN => 'base_info:name', VERSIONS => 4}   查看多个版本的数据

5.查看表的名称：
list

6.Hbase中数据库的概念，namespaces

    创建一个命名空间my_ns
    create_namespace 'my_ns'
    在命名空间my_ns里创建my_table
    create 'my_ns:my_table', 'fam'
    drop namespace
    drop_namespace 'my_ns'

    alter namespace
    alter_namespace 'my_ns', {METHOD => 'set', 'PROPERTY_NAME' => 'PROPERTY_VALUE'}
    默认的namespace：
       default:一般没有定义名称空间的表都在default里面。
       hbase:里面存放的是Hbase的基础表（root,meta,namespace）
	
	

