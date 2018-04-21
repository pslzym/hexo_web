---
title: Storm事务示例程序分析
tags: |-

  - storm
permalink: stormshi-wu-shi-li-cheng-xu-fen-xi
id: 36
updated: '2016-10-08 14:56:28'
date: 2016-10-08 14:18:57
---

##Storm事务示例程序分析

* MyMata.java 原始数据类

```
public class MyMata implements Serializable {

    private long beginPoint; // 事务开始位置
    private int num;
    private static final long serialVersionUID = 1L;

    @Override
    public String toString() {
        return "beginPoint : " + getBeginPoint() + "---- len : " + getNum();
    }

    public void setBeginPoint(long beginPoint) {
        this.beginPoint = beginPoint;
    }

    public void setNum(int num) {
        this.num = num;
    }

    public long getBeginPoint() {
        return beginPoint;
    }

    public int getNum() {
        return num;
    }
}

```

MyTxSpout.java  spout类，产生原始数据。

```
public class MyTxSpout implements ITransactionalSpout<MyMata> {

    Map<Long, String> dbMap = null;
    //private static final long serialVersionUID = 1L;

    public MyTxSpout(){
        Random random = new Random();

        dbMap = new HashMap<Long, String>();

        String[] hosts = { "www.taobao.com" };
        String[] session_id = { "ABYH6Y4V4SCVXTG6DPB4VH9U123", "XXYH6YCGFJYERTT834R52FDXV9U34", "BBYH61456FGHHJ7JL89RG5VV9UYU7",
                "CYYH6Y2345GHI899OFG4V9U567", "VVVYH6Y4V4SFXZ56JIPDPB4V678" };
        String[] time = { "2014-01-07 08:40:50", "2014-01-07 08:40:51", "2014-01-07 08:40:52", "2014-01-07 08:40:53",
                "2014-01-07 09:40:49", "2014-01-07 10:40:49", "2014-01-07 11:40:49", "2014-01-07 12:40:49" };

        for (long i = 0; i < 100; i++)
            dbMap.put(i, hosts[0] + "\t" + session_id[random.nextInt(5)] + "\t" + time[random.nextInt(8)]);
    }

    public Coordinator<MyMata> getCoordinator(Map map, TopologyContext topologyContext) {

        return new MyCoordinator(); ///用于发送和保存事务相关信息
    }

    public Emitter<MyMata> getEmitter(Map map, TopologyContext topologyContext) {
        return new MyEmitter(dbMap);//用于发送每一个事务具体的数据
    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("tx", "log"));
    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }
}
```

* MyCoordinator.java   //用于为每一次事务设定原始数据，并启动事务

```
public class MyCoordinator implements ITransactionalSpout.Coordinator<MyMata> {

    private static int BATCH_NUM = 10;

    public MyMata initializeTransaction(BigInteger bigInteger, MyMata preMyMata) {

        long beginPoint = 0;
        if (preMyMata == null){
            beginPoint = 0;
        }else {
            beginPoint = preMyMata.getBeginPoint() + preMyMata.getNum();
        }

        MyMata nowMyMata = new MyMata();
        nowMyMata.setBeginPoint(beginPoint);
        nowMyMata.setNum(BATCH_NUM);
        System.err.println("MyCoordinator initializeTransaction 启动一个事务: " + nowMyMata.toString());

        return nowMyMata;
    }

    public boolean isReady() {
        Utils.sleep(2000);
        return true;
    }

    public void close() {

    }
}
```

* MyEmitter.java   //获取Coordinator里面设定的原始数据，并根据相关类容发送相关tuple

```
public class MyEmitter implements ITransactionalSpout.Emitter<MyMata> {

    Map<Long, String> dbMap = null;

    public MyEmitter(Map<Long, String> dbMap){
        this.dbMap = dbMap;
    }

    public void emitBatch(TransactionAttempt transactionAttempt, MyMata myMata, BatchOutputCollector batchOutputCollector) {
        long beginPoint = myMata.getBeginPoint();
        int num = myMata.getNum();

        for(long i = beginPoint; i < beginPoint + num; i++){
            if (dbMap.get(i) == null){
                continue;
            }
            batchOutputCollector.emit(new Values(transactionAttempt, dbMap.get(i)));
        }
    }

    public void cleanupBefore(BigInteger bigInteger) {

    }

    public void close() {

    }
}

```

* MyTransactionalBolt.java //一级bolt。写代码的时候注意，每一次batch事务过来后，都会重新生成新德MyTransactionalBolt类。类里面相关的临时数据会全部消失。

```
public class MyTransactionalBolt extends BaseTransactionalBolt {

    private TransactionAttempt tx = null;
    private BatchOutputCollector batchOutputCollector = null;
    //private static final long serialVersionUID = 1L;

    int count = 0;

///一次批处理事务会调用一次
    public void prepare(Map map, TopologyContext topologyContext, BatchOutputCollector batchOutputCollector,
                        TransactionAttempt transactionAttempt) {
        this.tx = transactionAttempt;
        this.batchOutputCollector = batchOutputCollector;

        System.err.println("MyTransactionalBolt prepare  TransactionId: " + transactionAttempt.getTransactionId() +
                "---- AttemptId:" + transactionAttempt.getAttemptId());
    }

////对每一个tuple都会调用一次
    public void execute(Tuple tuple) {

        TransactionAttempt txExe = (TransactionAttempt)tuple.getValueByField("tx");

        System.err.println("MyTransactionalBolt execute  TransactionId: " + txExe.getTransactionId() +
                "---- AttemptId:" + txExe.getAttemptId());

        String log = tuple.getStringByField("log");

        if (log != null && log.length() > 0){
            count++;
        }
    }

////一次批处理事务会调用一次
    public void finishBatch() {
        System.err.println("MyTransactionalBolt finishBatch : " + count);
        this.batchOutputCollector.emit(new Values(tx, count));
    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("tx", "count"));
    }
}
```

*  MyCommitter.java   //批处理会按照事务的顺序进行committer

```
public class MyCommitter extends BaseTransactionalBolt implements ICommitter {

    //private static final long serialVersionUID = 1L;

    int sum = 0;
    TransactionAttempt tx = null;
    BatchOutputCollector batchOutputCollector = null;

    public static final String GLOBAL_KEY = "GLOBAL_KEY";
    ///这里是静态的，不管来多少次事务，里面的相关数据都会保存。
    public static Map<String, DbValue> dbMap = new HashMap<String, DbValue>() ;
    //public Map<String, DbValue> dbMap = new HashMap<String, DbValue>() ;

    public void prepare(Map map, TopologyContext topologyContext, BatchOutputCollector batchOutputCollector,
                        TransactionAttempt transactionAttempt) {
        this.tx = transactionAttempt;
        this.batchOutputCollector = batchOutputCollector;

        System.err.println("MyCommitter prepare : " + tx.getTransactionId() + " : " + tx.getAttemptId());
    }

    public void execute(Tuple tuple) {
        sum += tuple.getInteger(1);
        System.err.println("MyCommitter execute : " + tx.getTransactionId() + " : " + tx.getAttemptId());
    }

////事务处理完成后，会按照顺序来进行提交
    public void finishBatch() {
        System.err.println("MyCommitter finishBatch : " + tx.getTransactionId() + " : " + tx.getAttemptId());
        DbValue value = dbMap.get(GLOBAL_KEY);
        DbValue newValue;

        if (value == null || !value.txid.equals(tx.getTransactionId())){
            newValue = new DbValue();
            newValue.txid = tx.getTransactionId();
            if (value == null){
                newValue.count = sum;
            }else {
                newValue.count = sum + value.count;
            }
            dbMap.put(GLOBAL_KEY, newValue);
        }

        System.out.println("total==========================:"+dbMap.get(GLOBAL_KEY).count);
    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {

    }

    public static class DbValue{
        BigInteger txid;
        int count = 0;
    }
}
```

* MyTopo.java   ///事务top的相关编写

```
public class MyTopo {

    public static void main(String [] args){
        TransactionalTopologyBuilder builder = new TransactionalTopologyBuilder("ttbId","spoutid",new MyTxSpout(),1);
        builder.setBolt("bolt1", new MyTransactionalBolt(),3).shuffleGrouping("spoutid");
        builder.setBolt("committer", new MyCommitter(),1).shuffleGrouping("bolt1") ;

        Config conf = new Config() ;
        conf.setDebug(true);

        if (args.length > 0) {
            try {
                StormSubmitter.submitTopology(args[0], conf, builder.buildTopology());
            } catch (AlreadyAliveException e) {
                e.printStackTrace();
            } catch (InvalidTopologyException e) {
                e.printStackTrace();
            } catch (AuthorizationException e) {
                e.printStackTrace();
            }
        }else {
            LocalCluster localCluster = new LocalCluster();
            localCluster.submitTopology("mytopology", conf, builder.buildTopology());
        }

    }
}
```


