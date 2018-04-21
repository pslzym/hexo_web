---
title: Storm示例程序
tags: |-

  - storm
permalink: stormshi-li-cheng-xu
id: 31
updated: '2016-10-05 16:10:00'
date: 2016-10-05 16:04:21
---

##Storm示例程序
*	测试数据
*	Spout程序
* 	Bolt程序
* 主程序

###测试数据
由于是简单的示例测试程序，我直接弄了一个代码生成了相关的数据文件。
最后文件数据的格式如下：
<code>
www.pslland.org	CYYH6Y2345GHI899OFG4V9U567	2014-01-07 11:40:49
www.taobao.com	XXYH6YCGFJYERTT834R52FDXV9U34	2014-01-07 08:40:50
www.taobao.com	CYYH6Y2345GHI899OFG4V9U567	2014-01-07 09:40:49
www.taobao.com	BBYH61456FGHHJ7JL89RG5VV9UYU7	2014-01-07 08:40:53
www.taobao.com	VVVYH6Y4V4SFXZ56JIPDPB4V678	2014-01-07 10:40:49
www.taobao.com	ABYH6Y4V4SCVXTG6DPB4VH9U123	2014-01-07 08:40:50
www.taobao.com	CYYH6Y2345GHI899OFG4V9U567	2014-01-07 11:40:49
www.pslland.org	XXYH6YCGFJYERTT834R52FDXV9U34	2014-01-07 08:40:52
</code>
行数自己控制

```
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Random;

/**
 * Created by psl on 9/24/16.
 */
public class genrateData {
    public static void main(String [] argv){
        File logFile = new File("test.log");

        Random random = new Random();

        String [] hosts = {"www.pslland.org", "www.taobao.com"};

        String [] sessions = {"ABYH6Y4V4SCVXTG6DPB4VH9U123", "XXYH6YCGFJYERTT834R52FDXV9U34", "BBYH61456FGHHJ7JL89RG5VV9UYU7",
                "CYYH6Y2345GHI899OFG4V9U567", "VVVYH6Y4V4SFXZ56JIPDPB4V678"};
        String [] time = {"2014-01-07 08:40:50", "2014-01-07 08:40:51", "2014-01-07 08:40:52", "2014-01-07 08:40:53",
                "2014-01-07 09:40:49", "2014-01-07 10:40:49", "2014-01-07 11:40:49", "2014-01-07 12:40:49"};


        StringBuffer stringBuffer = new StringBuffer();

        for (int i = 0; i < 50; i++){
            stringBuffer.append(hosts[random.nextInt(2)] + '\t' + sessions[random.nextInt(5)] + '\t' + time[random.nextInt(8)] + '\n');
        }

        if (!logFile.exists()){
            try {
                logFile.createNewFile();
            }catch (IOException e) {
                e.printStackTrace();
            }
        }
        FileOutputStream fs = null;
        byte [] b = (stringBuffer.toString()).getBytes();
        try {
            fs = new FileOutputStream(logFile);
            fs.write(b);
            fs.close();
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

###Spout程序

![](/uploads/2016/10/spout1.png)
类继承图

![](/uploads/2016/10/spout2.png)
ISpout接口

Spout程序层抽象是ISpout接口
Open()是初始化方法
nextTuple()循环发射数据
ack() 成功处理tuple回调方法
Fail()处理失败tuple回调方法
activate和deactivate ：spout可以被暂时激活和关闭
close方法在该spout关闭前执行，
但是并不能得到保证其一定被执行，kill -9时不执行，
Storm kill {topoName} 时执行

原则：通常情况下（Shell和事务型的除外），实现一个Spout，可以直接实现接口IRichSpout，如果不想写多余的代码，可以直接继承BaseRichSpout。

```
import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichSpout;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Values;

import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.InputStreamReader;
import java.util.Map;

/**
 * Created by psl on 9/24/16.
 */
public class mySpout implements IRichSpout{
    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("LOG"));
    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }

    SpoutOutputCollector spoutOutputCollector = null;
    FileInputStream fs = null;
    InputStreamReader isr = null;
    BufferedReader br;

    public void open(Map map, TopologyContext topologyContext, SpoutOutputCollector spoutOutputCollector) {

        try {
            this.spoutOutputCollector = spoutOutputCollector;
            this.fs = new FileInputStream("test.log");
            this.isr = new InputStreamReader(fs, "UTF-8");
            this.br = new BufferedReader(isr);

        }catch (Exception e){
            e.printStackTrace();
        }

    }

    public void close() {
        try {
            this.br.close();
            this.isr.close();
            this.fs.close();
        }catch (Exception e){
            e.printStackTrace();
        }

    }

    public void activate() {

    }

    public void deactivate() {

    }

    String str = null;
    public void nextTuple() {
        try {
            while ((str = this.br.readLine()) != null){
                spoutOutputCollector.emit(new Values(str));
                Thread.sleep(3000);
            }
        }catch ( Exception e){
            e.printStackTrace();
        }

    }

    public void ack(Object o) {
        System.out.println("spout ack: " + o.toString());

    }

    public void fail(Object o) {
        System.out.println("spout fail: " + o.toString());
    }
}
```


###Bolt程序

![](/uploads/2016/10/bolt1.png)
Bolt类继承图

![](/uploads/2016/10/bolt2.png)
IBolt方法

prepare方法进行初始化，
传入当前执行的上下文
execute接受一个tuple进行处理，
也可emit数据到下一级组件
cleanup 同ISpout的close方法，在关闭前调用，
不保证其一定执行。

IBolt继承了Serializable，我们在nimbus上提交了topology以后，创建出来的bolt会序列化后发送到具体执行的worker(工作进程)上去。worker在执行该Bolt时，会先调用prepare方法传入当前执行的上下文.
execute接受一个tuple进行处理，并用prepare方法传入的OutputCollector的ack方法（表示成功）或fail（表示失败）来反馈处理结果.还可以通过OutputCollector的emit方法把结果发射到下一级组件。

IBasicBolt接口，实现该接口的Bolt不用在代码中提供反馈结果了，Storm内部会自动反馈成功。如果你确实要反馈失败，可以抛出FailedException。

（后续章节讲到的Trident均不用显性调用ack和fail，框架会自动调。）

实现一个Bolt，可以实现IRichBolt接口或继承BaseRichBolt，如果不想自己处理结果反馈，可以实现 IBasicBolt接口或继承BaseBasicBolt，它实际上相当于自动做了prepare方法和 collector.emit.ack(inputTuple)。

```
import org.apache.storm.task.IBolt;
import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichBolt;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;

import java.util.Map;

/**
 * Created by psl on 9/24/16.
 */
public class mybolt implements IRichBolt{

    OutputCollector outputCollector = null;
    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        this.outputCollector = outputCollector;
    }

    String valueString = null;
    int num = 0;
    public void execute(Tuple tuple) {
        try {
            valueString = tuple.getStringByField("LOG");
            if(valueString != null){
                num++;
                System.err.println(Thread.currentThread().getName() + "   line : " + num + "   sessionId : " + valueString.split("\t")[1]);
            }
            outputCollector.ack(tuple);
            Thread.sleep(2000);
        }catch (Exception e){
            outputCollector.fail(tuple);
            e.printStackTrace();
        }

    }

    public void cleanup() {

    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields(""));

    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }
}
```

###主程序

```
import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.StormSubmitter;
import org.apache.storm.topology.TopologyBuilder;

/**
 * Created by psl on 9/24/16.
 */
public class main {
    public static void main(String [] argv){
        TopologyBuilder topologyBuilder = new TopologyBuilder();

        topologyBuilder.setSpout("spout", new mySpout(), 1);
        topologyBuilder.setBolt("bolt", new mybolt(), 1).shuffleGrouping("spout");

        Config conf = new Config() ;
        conf.setDebug(true);

        if (argv.length > 0){
            try {
                StormSubmitter.submitTopology(argv[0], conf, topologyBuilder.createTopology());

            }catch (Exception e){
                e.printStackTrace();
            }
        }else {
            LocalCluster localCluster = new LocalCluster();
            localCluster.submitTopology("mytopology", conf, topologyBuilder.createTopology());

        }
    }

}

```

