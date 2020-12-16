```java
// 创建StreamingContext
SparkConf conf = new SparkConf().setMaster("local[2]").setAppName("NetworkWordCount");
// 批处理间隔为10秒
JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(10));
// 创建一个监听localhost:1999的DStream
JavaReceiverInputDStream<String> lines = jssc.socketTextStream("localhost", 1999);

JavaDStream<String> words = lines.flatMap(x -> Arrays.asList(x.split(" ")).iterator());

JavaPairDStream<String, Integer> pairs = words.mapToPair(s -> new Tuple2<>(s, 1));
JavaPairDStream<String, Integer> wordCounts = pairs.reduceByKey(Integer::sum);

wordCounts.print();

// 开始计算
jssc.start();
// 等待计算结束
jssc.awaitTermination();
```

使用nc命令向localhost:1999输入单词
```shell
nc -lk 9999
> a
> b
> c
> a
> a
# Time: 1608120570000 ms
# 20/12/16 12:09:30 INFO TaskSchedulerImpl: Removed TaskSet 8.0, whose tasks have all completed, from pool
# -------------------------------------------
# (b,1)
# (a,3)
# (c,1)
```

#### DStream
![spark003](http://git.nuozhilin.site/luzhong/images/raw/branch/master/spark003.png)
![spark004](http://git.nuozhilin.site/luzhong/images/raw/branch/master/spark004.png)

Spark Streaming使用DStream（离散化流）作为数据量的基本抽象，DStream是随时间推移而收到的数据的序列。在内部每个时间区间收到的数据都作为RDD存在，而DStream是由这些RDD所组成的序列。

#### 无状态转化操作
#### 有状态转化操作
UpdateStateByKey用于保存历史记录，有时我们需要在DStream中跨批次维护状态。针对这种情况updateStateByKey()为我们提供了对一个状态变量的访问。用于键值对形式的DStream，给定一个由(键，事件)对构成的DStream，并传递一个指定如何根据新的事件更新每个键对应状态的函数，它可以构建出一个新的DStream其内部数据为(键，状态)对。
#### 窗口操作
![spark002](http://git.nuozhilin.site/luzhong/images/raw/branch/master/spark002.png)

可以设置窗口的大小和滑动窗口的间隔来动态的获取当前Steaming不同时间区间的状态数据。
所有基于窗口的操作都需要两个参数，分别为windowLength（窗口时长）以及slideInterval（滑动步长），两者都必须是 StreamContext的批次间隔的整数倍。窗口时长控制每次计算最近的多少个批次的数据，如果有一个以10秒为批次间隔的源DStream，要创建一个最近30秒的时间窗口(即最近3个批次)，就应当把windowLength设为30秒。而滑动步长的默认值与批次间隔相等，用来控制对新的DStream进行计算的间隔。如果源DStream批次间隔为10秒，并且我们只希望每两个批次计算一次窗口结果就应该把slideInterval设置为20秒。