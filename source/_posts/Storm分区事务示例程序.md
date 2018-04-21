---
title: Storm分区事务示例程序
tags: |-

  - storm
permalink: stormfen-qu-shi-wu-shi-li-cheng-xu
id: 37
updated: '2016-10-08 15:55:27'
date: 2016-10-08 15:49:40
---

##Storm分区事务示例程序

分区事务和事务相关代码的区别主要在于spout端的代码

* MyPtTxSpout.java   //分区事务的spout端代码

```
public Map<Integer, Map<Long,String>> PT_DATA_MP = new HashMap<Integer, Map<Long,String>>();
    public static int BATCH_NUM = 10 ;

    public MyPtTxSpout()
    {
        Random random = new Random();
        String[] hosts = { "www.taobao.com" };
        String[] session_id = { "ABYH6Y4V4SCVXTG6DPB4VH9U123", "XXYH6YCGFJYERTT834R52FDXV9U34", "BBYH61456FGHHJ7JL89RG5VV9UYU7",
                "CYYH6Y2345GHI899OFG4V9U567", "VVVYH6Y4V4SFXZ56JIPDPB4V678" };
        String[] time = { "2014-01-07 08:40:50", "2014-01-07 08:40:51", "2014-01-07 08:40:52", "2014-01-07 08:40:53",
                "2014-01-07 09:40:49", "2014-01-07 10:40:49", "2014-01-07 11:40:49", "2014-01-07 12:40:49" };

        for (int j = 0; j < 5; j++) {
            HashMap<Long, String> dbMap = new HashMap<Long, String> ();
            for (long i = 0; i < 100; i++) {
                dbMap.put(i,hosts[0]+"\t"+session_id[random.nextInt(5)]+"\t"+time[random.nextInt(8)]);
            }
            PT_DATA_MP.put(j, dbMap);
        }

    }

    public Coordinator getCoordinator(Map map, TopologyContext topologyContext) {
        return new MyCoordinator();
    }

    public Emitter<MyMata> getEmitter(Map map, TopologyContext topologyContext) {
        return new MyEmitter();
    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {
        outputFieldsDeclarer.declare(new Fields("tx", "log"));
    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }

    public class MyCoordinator implements  IPartitionedTransactionalSpout.Coordinator{

        public int numPartitions() {
            ///返回有几个分区
            return 5;
        }

        public boolean isReady() {
            Utils.sleep(2000);
            return true;
        }

        public void close() {

        }
    }

    public class MyEmitter implements IPartitionedTransactionalSpout.Emitter<MyMata>{

        public MyMata emitPartitionBatchNew(TransactionAttempt transactionAttempt,
                                            BatchOutputCollector batchOutputCollector, int partition, MyMata myMata) {
            ///新的事务调用的发送函数
            //此函数可以和非分区的事务代码中 Coordinator做对比   基本差不多
            long beginPoint = 0;
            if(myMata == null){
                beginPoint = 0;
            }else{
                beginPoint = myMata.getBeginPoint() + myMata.getNum();
            }

            MyMata myMata1 = new MyMata();
            myMata1.setNum(BATCH_NUM);
            myMata1.setBeginPoint(beginPoint);

            //启动一个事务
            emitPartitionBatch(transactionAttempt, batchOutputCollector, partition, myMata1);

            return myMata1;
        }

        public void emitPartitionBatch(TransactionAttempt transactionAttempt,
                                       BatchOutputCollector batchOutputCollector, int partition, MyMata myMata) {
            //失败的事务调用的函数
            long beginPoint = myMata.getBeginPoint() ;
            int num = myMata.getNum() ;

            Map<Long, String> batchMap = PT_DATA_MP.get(partition);
            for (long i = beginPoint; i < num+beginPoint; i++) {
                if (batchMap.get(i)==null) {
                    break;
                }
                batchOutputCollector.emit(new Values(transactionAttempt,batchMap.get(i)));
            }
        }

        public void close() {

        }
    }
```



