---
title: Hbase协处理器
tags: |-

  - Hbase
permalink: hbasexie-chu-li-qi
id: 30
updated: '2016-10-04 18:24:02'
date: 2016-10-04 18:23:00
---

##Hbase 协处理器
HBase 做为列数据库最令人诟病的特性包括：
无法轻易建立‘二级索引’，难以执行求和、计数、排序等操作。

为了减少在客服端做大量的运算，Hbase引入了coprocessor，能够轻易建立二次索引、复杂过滤器以及访问控制符等。

协处理器分为两种类型，系统协处理器可以全局导入region server上所有的数据表，表协处理器即是用户可以指定一张表使用协处理器。
协处理器为了提高灵活性，提供了两种不同的插件。一个是观察者模式（observer），类似与数据库的触发器。另一个是终端（endpoint）,动态的终端有点像存储过程。

* observer
* endpoint

###observer
![](/uploads/2016/10/60b135e5-04c6-4197-b262-e7cd08de784b.png)
模型

以上的函数是比较老的，注意最新的代码。

Observer最新的类型有四种：

* RegionObserver: 提供客户端的数据操纵事件钩子:Get、Put、Delete、Scan等。

* RegionServerObserver : A RegionServerObserver allows you to observe events related to the RegionServer’s operation, such as starting, stopping, or performing merges, commits, or rollbacks.  最新加的。

* MasterObserver： 提供DDL-类型的操作钩子。如创建、删除、修改数据表等。 这些接口可以同时使用在同一个地方,按照不同优先级顺序执行.用户可以任意基于协处理 器实现复杂的HBase功能层。HBase有很多种事件可以触发观察者方法,这些事件与方法 从HBase0.92版本起,都会集成在HBase API中。不过这些API可能会由于各种原因有所改 动,不同版本的接口改动比较大。

* WalObserver：提供WAL相关操作钩子。

主要讨论RegionObserver相关代码的编写：

	public class RegionObserverTest extends BaseRegionObserver{

    private static byte[] fixed_rowkey = Bytes.toBytes("0004");

    @Override
    public void preGetOp(ObserverContext<RegionCoprocessorEnvironment> e, Get get, List<Cell> results) throws IOException {
        if(Bytes.equals(get.getRow(), fixed_rowkey)){
            KeyValue keyValue = new KeyValue(get.getRow(), Bytes.toBytes("time"), Bytes.toBytes("key_changed"),
                    Bytes.toBytes("value_changed"));

            results.add(keyValue);

        }
    }
	}

	在编写代码中遇到的问题：
		1.IDEA找不到BaseRegionObserver。这个是因为原先我们编写Hbase客服端程序只用到了hbase-client包，在编写coprocessor程序的时候需要导入hbase-server.
		\<dependency>
	            \<groupId>org.apache.hbase\</groupId>
	            \<artifactId>hbase-server\</artifactId>
	            \<version>1.2.3\</version>
	  \</dependency>
	  2.程序正常导入表之后，scan出来的行没有变化。这个是因为此coprocessor只是针对get方法，使用get去获取某一行就OK。
	  
如何使其正常运作。

1.	先将上述的代码打包生成一个jar文件。
2.	将jar文件传入HDFS的一个相应的路径
3. disable 表 （disable 'user_info'）
4. alter 修改表相关配置 （alter 'user_info', 'coprocessor'=>'hdfs:///coprocessor/hbaseRegionObserver.jar|org.pslland.RegionObserverTest||'）
5. enable相关表   （enable 'user_info'）
6. get相关数据，检查是否生效    （get 'user_info', '0004'）

###endpoint
endpoint常用于求和等高级功能，相关API在1.2.3里面改动比较大。相关例子参考：
http://www.3pillarglobal.com/insights/hbase-coprocessors



