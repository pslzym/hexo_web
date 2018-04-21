---
title: Storm不透明事务示例
tags: |-

  - storm
permalink: stormbu-tou-ming-shi-wu-shi-li
id: 38
updated: '2016-10-08 16:56:07'
date: 2016-10-08 16:55:37
---

##Storm不透明事务示例
在IPartitionedTransactionalSpout 案例基础上用IOpaquePartitionedTransactionalSpout 实现
实现分析：
需调整Spout端 和  Bolt端
Bolt端主要是调整写库部分

class DbValue {
String dateStr;  // 按日期汇总
int count = 0;  // 汇总数
BigInteger txid; // 事务ID
Int prev_count ; // 上个事务批次处理后的结果
} 
DbValue模拟一个数据表

对于commiter bolt必须要记录上一次事务tuple的个数，因为可能同一个事务重发之后

* MyOpaquePtTxSpout.java   

```
public class MyOpaquePtTxSpout implements IOpaquePartitionedTransactionalSpout {

    public static int BATCH_NUM = 10 ;
    public Map<Integer, Map<Long,String>> PT_DATA_MP = new HashMap<Integer, Map<Long,String>>();

    public MyOpaquePtTxSpout()
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


    public Emitter getEmitter(Map map, TopologyContext topologyContext) {
        return new MyEmiiter();
    }

    public Coordinator getCoordinator(Map map, TopologyContext topologyContext) {
        return new MyCoordinator();
    }

    public void declareOutputFields(OutputFieldsDeclarer outputFieldsDeclarer) {

    }

    public Map<String, Object> getComponentConfiguration() {
        return null;
    }

    public class MyCoordinator implements IOpaquePartitionedTransactionalSpout.Coordinator{

        public boolean isReady() {
            Utils.sleep(2000);
            return true;
        }

        public void close() {

        }
    }

    public class MyEmiiter implements IOpaquePartitionedTransactionalSpout.Emitter<MyMata>{

        public MyMata emitPartitionBatch(TransactionAttempt transactionAttempt,
                                         BatchOutputCollector batchOutputCollector, int partition, MyMata myMata) {
            // TODO Auto-generated method stub
            System.err.println("emitPartitionBatch partition:"+partition);
            long beginPoint = 0;
            if (myMata == null) {
                beginPoint = 0 ;
            }else {
                beginPoint = myMata.getBeginPoint() + myMata.getNum() ;
            }

            MyMata mata = new MyMata() ;
            mata.setBeginPoint(beginPoint);
            mata.setNum(BATCH_NUM);
            System.err.println("启动一个事务："+mata.toString());

            Map<Long, String> batchMap = PT_DATA_MP.get(partition);
            for (Long i = mata.getBeginPoint(); i < mata.getBeginPoint()+mata.getNum(); i++) {
                if (batchMap.size()<=i) {
                    break;
                }
                batchOutputCollector.emit(new Values(transactionAttempt, batchMap.get(i)));
            }
            return mata;
        }

        public int numPartitions() {
            return 5;
        }

        public void close() {

        }
    }
}
```


