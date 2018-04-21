---
title: Trident 理论学习
tags: |-

  - storm
permalink: trident-li-lun-xue-xi
id: 39
updated: '2016-10-09 17:24:34'
date: 2016-10-08 20:47:44
---

##Trident 理论学习

###Trident API -- Spout
ITridentSpout :最通用的Spout,可以支持事务或者不透明事务语义。
IBatchSpout : 一个非事务spout。
IPartitionedTridentSpout: 分区事务spout，从数据源（eg:Kafka集群）读分区数据。
IOpaquePartitionedTridentSpout: 不透明分区事务spout，从数据源读分区数据。


###Trident API  -- Bolt
唯一显性Bolt接口：ITridentBatchBolt，但很少用。
Trident编程特点就是Stream。
Trident的topology会被编译成尽可能高效的Storm topology。只有在需要对数据进行repartition的时候（如groupby或者shuffle）才会把tuple通过network发送出去.

###Trident 概念之Operation
Operation 类相关概念：

 * Function ， 如BaseFunction, 如TridentWordCount 中用的split、如BaseQueryFunction, TridentWordCount中用的MapGet,从State中查询
 * Filter : 如BaseFilter， 如TridentWordCount 中用的FilterNull
 * 聚合类： 
 			CombinerAggregator<T>
 			Aggregator<T>
 			ReducerAggregator<T>
 			
 Function:
 
```
 public class MyFunction extends BaseFunction {
    public void execute(TridentTuple tuple, TridentCollector collector) {
        for(int i=0; i < tuple.getInteger(0); i++) {
            collector.emit(new Values(i));
        }
    }
}
```

假设有一个叫“mystream”输入流有【“a”，“b”，“c”】三个字段
【1，2，3】
【4，1，6】
【3，0，8】
 运行mystream.each(new Fields("b"), new MyFunction(), new Fields("d"))
 运行的结果将会有四个字段【“a”, "b", "c", "d"】
 [1,2,3,0]
 [1,2,3,1]
 [4,1,6,0]
 
 Filters:
 Filters接收一个元组（tuple），决定是否需要继续保留这个元组。比如：
 

```
public class MyFilter extends BaseFilter{
    public booleanisKeep(TridentTuple tuple) {
        return tuple.getInteger(0) == 1 && tuple.getInteger(1) == 2;
    }
}
```

假如有如下输入：
【1，2，3】
【2，1，1】
【2，3，4】
运行下面的代码：
mystream.each(new Fields("b", "a"), new MyFilter())
结果将会是：
【2，1，1】


聚合：  
例子：persistentAggregate(new MemoryMapState.Factory(), new Count(), new Fields("count"))   在持久化中使用到。new Count就是具体的聚合类。

聚合接口：
CombinerAggreator：在每个tuple上运行init，使用combine去联合结果 。如果批次没有数据，运行zero函数。
ReduceAggregator:在init()初始化的时候产生一个值，每个输入的元组在这个值的基础上进行迭代并输出一个单独的值
Aggregate：功能最强大
1.      Init函数在执行批次操作之前被调用，并返回一个state对象，这个对象将会会传入到aggregate和complete函数中。
2.      Aggregate会对批次中每个tuple调用，这个方法可以跟新state也可以发射（emit）tuple。
3.      当这个批次分区的数据执行结束后调用complete函数。
 aggregate和persistentAggregate函数对流做聚合。Aggregate在每个批次上独立运行，persistentAggregate聚合流的所有的批次并将结果存储下来。
 
 
``` 
public class Sum implements CombinerAggregator<Number> {
    public Sum() {
    }

    public Number init(TridentTuple tuple) {
        return (Number)tuple.getValue(0);
    }

    public Number combine(Number val1, Number val2) {
        return Numbers.add(val1, val2);
    }

    public Number zero() {
        return Integer.valueOf(0);
    }
	}

```


