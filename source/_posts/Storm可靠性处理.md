---
title: Storm可靠性处理
tags: |-

  - storm
permalink: stormke-kao-xing-chu-li
id: 34
updated: '2016-10-07 01:34:28'
date: 2016-10-07 00:56:31
---

##Storm可靠性处理

  * spout可靠性处理
  * bolt可靠性处理

###spout可靠性处理
以单词计算的例子为例子：

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
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

/**
 * Created by psl on 10/5/16.
 */
public class SentenceSpout extends BaseRichSpout {

    private SpoutOutputCollector spoutOutputCollector = null;
    private ConcurrentHashMap<UUID, Values> pending;

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
        this.pending = new ConcurrentHashMap<UUID, Values>();
    }

    public void nextTuple() {
        UUID msgId = UUID.randomUUID();
        ///发送时必须带上msgId
        this.spoutOutputCollector.emit(new Values(sentence[index]), msgId);

        this.pending.put(msgId, new Values(sentence[index]));

        index++;
        if(index >= sentence.length){
            index = 0;
        }
        Utils.sleep(1000);
    }

    @Override
    public void ack(Object msgId) {
        System.err.println("spout ack");
        this.pending.remove(msgId);

    }

    @Override
    public void fail(Object msgId) {
        this.spoutOutputCollector.emit(this.pending.get(msgId), msgId);
    }
}

```

###bolt可靠性处理

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
            //发送时必须带上 tuple
            this.outputCollector.emit(tuple, new Values(word));
        }

        this.outputCollector.ack(tuple);
        System.err.println("split ack");


    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("word"));
    }
}

```
后面接着的bolt必须全部都加上tuple向后发送，并且处理完成后，显示的调用ack方法。
