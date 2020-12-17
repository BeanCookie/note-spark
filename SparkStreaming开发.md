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

#### ``map``

```java
public <U> DStream<U> map(Function1<T,U> mapFunc)
```

- 含义: 通过mapFunc将集合中原来的T类型的元素映射成U类型的元素
#### ``mapPartitions``

```java
public <U> DStream<U> mapPartitions(Function1<Iterator<T>, Iterator<U>> mapPartFunc, boolean preservePartitioning)
```

- 含义: mapPartFunc调用次数与分区数相同

#### ``flatMap``

```java
public <U> DStream<U> flatMap(Function1<T, TraversableOnce<U>> flatMapFunc)
```

- 含义: 将DStream的集合元素展平

#### ``reduce``

```java
public DStream<T> reduce(Function2<T, T, T> reduceFunc)
```

- 含义: 会返回一个单个元素的RDD流

#### ``union``

```java
public DStream<T> union(DStream<T> that)
```

- 含义: 将两个同类型DStream合并为一个新的DStream，前提是这两个DStream必须具有相同的批次间隔

#### ``transform``

```java
public <U> DStream<U> transform(Function1<RDD<T>, RDD<U>> transformFunc)
```

- 含义: transform算子是DStream所独有的，它用transformFunc函数从输入的RDD得到一个新的RDD，在transformFunc这个函数中，还可以使用RDD的算子、Spark SQL等操作，该算子在应对一些复杂操作时比较有用
```shell
distData.collect()
# Array[Int] = Array(1, 2, 3, 4, 5)

val mapDistData = distData.map(v => v * v)
mapDistData.collect()
# Array[Int] = Array(1, 4, 9, 16, 25)
```

#### ``transformWith``

```java
public <U,V> DStream<V> transformWith(DStream<U> other, Function2<RDD<T>, RDD<U>, RDD<V>> transformFunc)
```

- 含义: transformWith还可以完成与其他数据集进行join
```shell
distData.collect()
# Array[Int] = Array(1, 2, 3, 4, 5)

val mapDistData = distData.map(v => v * v)
mapDistData.collect()
# Array[Int] = Array(1, 4, 9, 16, 25)
```
#### 有状态转化操作

UpdateStateByKey用于保存历史记录，有时我们需要在DStream中跨批次维护状态。针对这种情况updateStateByKey()为我们提供了对一个状态变量的访问。用于键值对形式的DStream，给定一个由(键，事件)对构成的DStream，并传递一个指定如何根据新的事件更新每个键对应状态的函数，它可以构建出一个新的DStream其内部数据为(键，状态)对。
#### ``mapWithState``

```java
public <StateType, MappedType> MapWithStateDStream<K, V, StateType,MappedType> mapWithState(StateSpec<K, V, StateType, MappedType> spec)
```

- 含义: 
```shell

```

#### ``updateStateByKey``

```java
public <S> DStream<Tuple2<K,S>> updateStateByKey(Function4<Time, K, Seq<V>, Option<S>, scala.Option<S>> updateFunc, Partitioner partitioner, boolean rememberPartitioner, Option<RDD<Tuple2<K,S>>> initialRDD)
```

- 含义: 
```shell

```
#### 窗口操作
![spark002](http://git.nuozhilin.site/luzhong/images/raw/branch/master/spark002.png)

可以设置窗口的大小和滑动窗口的间隔来动态的获取当前Steaming不同时间区间的状态数据。
所有基于窗口的操作都需要两个参数，分别为windowLength（窗口时长）以及slideInterval（滑动步长），两者都必须是 StreamContext的批次间隔的整数倍。窗口时长控制每次计算最近的多少个批次的数据，如果有一个以10秒为批次间隔的源DStream，要创建一个最近30秒的时间窗口(即最近3个批次)，就应当把windowLength设为30秒。而滑动步长的默认值与批次间隔相等，用来控制对新的DStream进行计算的间隔。如果源DStream批次间隔为10秒，并且我们只希望每两个批次计算一次窗口结果就应该把slideInterval设置为20秒。
#### ``slice``

```java
public scala.collection.Seq<RDD<T>> slice(Time fromTime, Time toTime)
```

- 含义: 返回指定时间范围内的所有RDD

#### ``window``

```java
public DStream<T> window(Duration windowDuration, Duration slideDuration)
```

- 含义: 返回一个包含在此DStream上的滑动时间范围内看到的所有元素的DStream

#### ``reduceByWindow``

```java
public DStream<T> reduceByWindow(Function2<T, T, T> reduceFunc, Function2<T, T, T> invReduceFunc, Duration windowDuration, Duration slideDuration)
```

- 含义: 

#### ``countByWindow``

```java
public DStream<Object> countByWindow(Duration windowDuration, Duration slideDuration)
```

- 含义: 

#### ``countByValueAndWindow``

```java
public DStream<scala.Tuple2<T,Object>> countByValueAndWindow(Duration windowDuration, Duration slideDuration, int numPartitions, Ordering<T> ord)
```

- 含义: 