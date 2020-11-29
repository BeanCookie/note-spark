### 向Spark传递函数

实际开发项目时需要自定义大量的函数传递到RDD转换或者行动函数中, 因为所有代码都是通过Driver端分发到各个Executor中执行的, 因此会涉及到大量的RPC进行跨进程通信, 通信的数据都需要进行序列化处理, 通常推荐定义类对象和Lambda形式定义需要传递的函数。

```java

public class WordCountWithFunctionClass {
    private static final Pattern SPACE = Pattern.compile(" ");

    public static class SpaceLine implements FlatMapFunction<String, String> {

        @Override
        public Iterator<String> call(String line) throws Exception {
            return Arrays.asList(SPACE.split(line)).iterator();
        }
    }

    public static void main(String[] args) {

        if (args.length < 1) {
            System.err.println("Usage: JavaWordCount <file>");
            System.exit(1);
        }


        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount");
        SparkContext sparkContext = SparkContext.getOrCreate(sparkConf);

        JavaRDD<String> lines = sparkContext.textFile(args[0], 1).toJavaRDD();

      	// 使用类对象
        JavaRDD<String> words = lines.flatMap(new SpaceLine());
        // 使用匿名内部类
        JavaPairRDD<String, Integer> ones = words.mapToPair(new PairFunction<String, String, Integer>() {
            @Override
            public Tuple2<String, Integer> call(String word) throws Exception {
                return new Tuple2<>(word, 1);
            }
        });
        // 使用Lambda
        JavaPairRDD<String, Integer> counts = ones.reduceByKey(Integer::sum);

        List<Tuple2<String, Integer>> output = counts.collect();
        for (Tuple2<?,?> tuple : output) {
            System.out.println(tuple._1() + ": " + tuple._2());
        }
        sparkContext.stop();
    }
}
```

### Spark中的闭包

在Spark集群中执行代码变量和方法的生命周期和作用域与本地单机程序有很大的区别，如果不慎修改超过RDD作用范围的变量会引起数据混乱甚至异常。

```java
public class CounterCumulative {
    public static void main(String[] args) {
        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount");
        SparkContext sparkContext = SparkContext.getOrCreate(sparkConf);
        AtomicInteger counter = new AtomicInteger();
        JavaRDD<String> lines = sparkContext.textFile(args[0], 1).toJavaRDD();

        lines.foreach(x -> counter.incrementAndGet());

        System.out.println("counter: " + counter.intValue());
    }
}
```
上面的这段代码是不符合规范的，通常分布式情况下counter始终为0，即使在本地环境下counter被更新了也只是偶然情况，无法保证分布式环境下会实现相同的效果, 下面我们具体分析一下原因。
为了执行JOB，Spark将RDD操作的处理分解为多个Task，每个Task由Executor执行。在执行之前Spark会计算Task的闭包。闭包是Executor在RDD上进行计算的时候必须可见的那些变量和方法(在这种情况下是foreach()), 闭包会被序列化并发送给每个Executor。发送给每个Executor的闭包中的变量是副本，因此当foreach函数内引用计数器时，它不再是Driver节点上的计数器。Driver节点的内存中仍有一个计数器但该变量是Executor不可见的, 执行者只能看到序列化闭包的副本。因此计数器的最终值仍然为0，因为计数器上的所有操作都引用了序列化闭包内的值。在本地模式下，在某些情况下该foreach函数实际上将在与Driver相同的JVM内执行，并且会引用相同的原始计数器因此可能实际更新counter。通常情况下编写闭包(像循环, 局部定义的方法)时不应该使用某些全局状态。Spark不无法更新闭包外部引用的对象, 某些执行此操作的代码可能会在本地模式下工作，但这只是偶然的情况，此类代码在分布式模式下将无法按预期运行。如果需要某些全局聚合请使用累加器。