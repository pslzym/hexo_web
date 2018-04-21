---
title: Storm单词计数示例
tags: |-

  - storm
permalink: stormdan-ci-ji-shu-shi-li
id: 33
updated: '2016-10-07 00:32:21'
date: 2016-10-07 00:30:43
---

##Storm单词计数示例
* top代码
* spout代码
* bolt代码
* 遇到的bug

###top代码

```
package org.pslland;

import org.apache.storm.Config;
import org.apache.storm.LocalCluster;
import org.apache.storm.StormSubmitter;
import org.apache.storm.generated.AlreadyAliveException;
import org.apache.storm.generated.AuthorizationException;
import org.apache.storm.generated.InvalidTopologyException;
import org.apache.storm.topology.TopologyBuilder;
import org.apache.storm.tuple.Fields;
import org.apache.storm.utils.Utils;

/**
 * Created by psl on 10/6/16.
 */
public class WordCountTopology {

    private static final String SENTENCE_SPOUT_ID = "sentence-spout";
    private static final String SPLIT_BOLT_ID = "split-bolt";
    private static final String COUNT_BOLT_ID = "count-bolt";
    private static final String REPORT_BOLT_ID = "report-bolt";
    private static final String TOPOLOGY_NAME = "word-count-topology";

    public static void main(String [] args) throws InvalidTopologyException, AuthorizationException, AlreadyAliveException {
        SentenceSpout sentenceSpout = new SentenceSpout();
        SplitSentenceBolt splitSentenceBolt = new SplitSentenceBolt();
        WordCountBolt wordCountBolt = new WordCountBolt();
        ReportBolt reportBolt = new ReportBolt();

        TopologyBuilder builder = new TopologyBuilder();

        builder.setSpout(SENTENCE_SPOUT_ID, sentenceSpout);//, 2);

        builder.setBolt(SPLIT_BOLT_ID, splitSentenceBolt, 2).setNumTasks(4).shuffleGrouping(SENTENCE_SPOUT_ID);
        //builder.setBolt(SPLIT_BOLT_ID, splitSentenceBolt).shuffleGrouping(SENTENCE_SPOUT_ID);

        builder.setBolt(COUNT_BOLT_ID, wordCountBolt, 6).fieldsGrouping(SPLIT_BOLT_ID, new Fields("word"));
        //builder.setBolt(COUNT_BOLT_ID, wordCountBolt).fieldsGrouping(SPLIT_BOLT_ID, new Fields("word"));

        builder.setBolt(REPORT_BOLT_ID, reportBolt).globalGrouping(COUNT_BOLT_ID);

        Config config = new Config();
        //config.setNumWorkers(2);

        if (args.length == 0) {

            LocalCluster cluster = new LocalCluster();

            cluster.submitTopology(TOPOLOGY_NAME, config, builder.createTopology());
            //StormSubmitter.submitTopology(TOPOLOGY_NAME, config, builder.createTopology());

            Utils.sleep(20000);

            cluster.killTopology(TOPOLOGY_NAME);

            cluster.shutdown();
        }else {
            StormSubmitter.submitTopology(args[0], config, builder.createTopology());
        }



    }
}
```

###spout代码

```
package org.pslland;

import org.apache.storm.spout.SpoutOutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.IRichBolt;
import org.apache.storm.topology.IRichSpout;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichSpout;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Values;
import org.apache.storm.utils.Utils;


import java.util.Map;

/**
 * Created by psl on 10/5/16.
 */
public class SentenceSpout extends BaseRichSpout {

    private SpoutOutputCollector spoutOutputCollector = null;

    private String [] sentence = {
        "my dog has fleas",
            "I like cold beverages",
            "the dog ate my homework",
            "don't have a cow man",
            "i don't think i like fleas"
    };

    private  int index = 0;

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("sentence"));
    }

    public void open(Map map, TopologyContext topologyContext, SpoutOutputCollector spoutOutputCollector) {
        this.spoutOutputCollector = spoutOutputCollector;
    }

    public void nextTuple() {
        this.spoutOutputCollector.emit(new Values(sentence[index]));

        index++;
        if(index >= sentence.length){
            index = 0;
        }
        Utils.sleep(1000);
    }
}

```

###bolt代码
splitSentenceBolt.java

```
package org.pslland;

import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

import java.util.Map;

/**
 * Created by psl on 10/5/16.
 */
public class SplitSentenceBolt extends BaseRichBolt {

    private OutputCollector outputCollector = null;

    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        this.outputCollector = outputCollector;
    }

    public void execute(Tuple tuple) {
        String sentence = tuple.getStringByField("sentence");
        String [] words = sentence.split(" ");

        System.out.println("splitSentence : " + sentence);

        for (String word : words){
            this.outputCollector.emit(new Values(word));
        }

    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("word"));
    }
}
```

WordCountBolt.java

```
package org.pslland;

import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.tuple.Fields;
import org.apache.storm.tuple.Tuple;
import org.apache.storm.tuple.Values;

import java.util.HashMap;
import java.util.Map;

/**
 * Created by psl on 10/5/16.
 */
public class WordCountBolt extends BaseRichBolt {

    private OutputCollector outputCollector = null;
    private HashMap<String, Long> counts = null;

    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        this.outputCollector = outputCollector;
        this.counts = new HashMap<String, Long>();
    }

    public void execute(Tuple tuple) {
        String word = tuple.getStringByField("word");
        Long count = this.counts.get(word);
        if(count == null){
            count = 0L;
        }
        count++;
        this.counts.put(word, count);
        this.outputCollector.emit(new Values(word, count));

    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("word", "count"));
    }
}

```

ReportBolt.java

```
package org.pslland;

import org.apache.storm.task.OutputCollector;
import org.apache.storm.task.TopologyContext;
import org.apache.storm.topology.OutputFieldsDeclarer;
import org.apache.storm.topology.base.BaseRichBolt;
import org.apache.storm.tuple.Tuple;

import java.util.*;

/**
 * Created by psl on 10/5/16.
 */
public class ReportBolt extends BaseRichBolt {

    private HashMap<String, Long> counts = null;

    public void prepare(Map map, TopologyContext topologyContext, OutputCollector outputCollector) {
        this.counts = new HashMap<String, Long>();
    }

    public void execute(Tuple tuple) {
        String word = tuple.getStringByField("word");
        Long count = tuple.getLongByField("count");

        this.counts.put(word, count);

        //this.cleanup();

    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        //last bolt
    }

    public void cleanup(){
        System.out.println("----------- FINAL COUNTS -----------");
        List<String> keys = new ArrayList<String>();
        keys.addAll(this.counts.keySet());

        Collections.sort(keys);

        for (String key : keys){
            System.out.println(key + " : " + this.counts.get(key));
        }

        System.out.println("-----------------------------------");


    }
}

```

###遇到的bug
我的代码是通过IDEA打包的。相关maven配置：

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
```

    <groupId>org.pslland</groupId>
    <artifactId>stormWC</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-core</artifactId>
            <version>1.0.2</version>
            <!--<scope>provided</scope>-->
        </dependency>
    </dependencies>

</project>

</code>

本地调试的时候去掉<scope>provided</scope>标签。

在集群提交测试的时候如果编译没有加上provided 标签会产生如下错误：
Caused by: java.io.IOException: Found multiple defaults.yaml resources. 

集群提交命令：
./storm  jar 包名  类名  参数

