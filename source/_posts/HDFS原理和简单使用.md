---
title: HDFS原理和简单使用
tags: |-

  - Hadoop
permalink: hdfsgao-ke-yong-bu-shu-fang-an
id: 10
updated: '2016-10-09 18:06:20'
date: 2016-09-18 21:51:09
---

##HDFS原理和简单使用
* namenode
* datanode
* 读写流程

###namenode

Namenode 管理文件系统的命名空间。他维护着文件系统树及整棵树内的所有文件和目录。这些信息以两个文件的形式永久保存在本地的磁盘上：命名空间镜像文件和编辑日志文件。Namenode 也记录着每个文件中各个块所在的数据节点信息，但他并不是永久保存块的位置信息，因为这些信息会在系统启动的时候由数据节点重建。

高可用方案：namenode存在单点失效的问题。在实现中，配置了一对活动-备用的namenode。当活动namenode失效，备用的namenode就会接管它的工作任务并开始服务于来自客服端的请求，不会有任何明显的中断。

1.	namenode之间需要通过高可用的共享存储实现编辑日志的共享。
2.	datanode需要同时向两个namenode发送数据块处理报告，因为数据块的映射信息存储在namenode的内存中，而非磁盘。
3.	客服端需要使用特定的机制来处理namenode的失效问题，这一机制对用户是透明的。

扩展方案：联邦HDFS。在联邦环境下，每一个namenode维护一个名称空间卷，包括命名空间的元数据和在该命名空间下的文件的所有数据块和数据块池。

###datanode
Datanode 是文件系统的工作节点。它们需要存储并检索数据块，并且定期向namenode发送它们所存储的块的列表。

###读写流程

![](/uploads/2016/10/hdfs--1.png)

    HDFS 读数据流程一般先会创建文件系统的类，通过这个类去访问NameNode上面的相关数据。
    1.  通过DistributedFileSystem类从NameNode 上获取所需文件对应的块位置。
    2. 然后通过FSDataInputStream类读取块所在DataNode上面具体块的数据。
    3. 数据返回后进行相应的拼接，形成具体的文件。


![](/uploads/2016/10/hdfs--2.png)

    HDFS写流程和读流程类似，先通过文件系统类去namenode上获取创建文件的块信息。然后根据块信息去写入文件。

    1. 通过DistributedFileSystem类从NameNode 上获取创建文件对应的块信息。
    2.  然后通过FSDataInputStream类将信息写入某一个datanode中，第一个datanode会依此向后面的datanode写入相关的数据，并且最后一个datanode会依此将写入成功的信息返回给前面的datanode。
    3.  最后一个块写入完成后，返回相应的写入完成信息，关闭相关的流和资源。



###HDFS数据的完整性和压缩
完整性：
    1.HDFS会对写入的所有数据计算校验和，并在读取数据时验证校验和。它针对每个由io.bytes.per.checksum指定字节的数据计算校验和。默认情况下为512。由于CRC-32校验和是4个字节，所以存储的校验和的额外资源低于1%。

    2.每一个datanode也会在一个后台线程中运行一个DataBlockScanner，从而定期的验证存储在这个datanode上的所有数据块。
    3.主要操作类LocalFileSystem 和ChecksumFileSystem

压缩：

     1.	减少存储文件所需要的磁盘空间，加速数据在网络和磁盘上的传输。


序列化

    序列化是指将结构化对象转化为字节流以便在网络上传输或写到磁盘进行永久存储的过程。反序列化是指将字节流转回结构化对象的逆过程。
    出现领域：进程间的通信和永久存储
    hadoop使用的是自己的序列化格式Writable。


###HDFS与POSIX接口不同之处

1.   新建一个文件之后，它能在文件系统的命名空间中立即可见。但是写入文件的内容并不保证能立即可见，即使数据流已经刷新并存储。当写入的数据超过一个块后，第一个数据块对新的reader就是可见的。之后的数据块也不例外。总之当前正在写入的块对其他的reader不可见。
2.   HDFS提供一个方法来使所有的缓存与数据节点强行同步，即对FSDataOutputStream调用sync()方法。（该方法在hadoop的后期版本中已经被替换为hsync 、hflush）


###相关API的使用

```

FileSystem.get(URI.create(uri), conf)   获得文件系统的操作句柄
（FSDataInputStream）InputStream in = fs.open(new Path(uri)) 获得文件句柄
IOUtils.copyBytes(in, System.out, 4096, false)  拷贝相关的数据
In.seek()   移动文件的位置
fs.create()创建文件
FSDataInputStream   操作输入流
FSDataOutputStream	操作输出流
Fs.mkdirs创建文件夹
Fs.delete()删除数据
文件夹操作：
FileStatus: 封装了文件系统中文件和目录的元数据，包括文件的长度、块大小、副本、修改时间、所有者以及权限信息。

```



