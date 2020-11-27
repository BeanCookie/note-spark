实际开发项目时需要自定义大量的函数传递到RDD转换或者行动函数中, 因为所有代码都是通过Driver端分发到各个Executor中执行的, 因此会涉及到大量的RPC进行跨进程通信, 通信的数据都需要进行序列化处理, 通常推荐两者形式定义需要传递的函数。

1. 匿名内部类
2. Lambda表达式
```java
public class WordCount {
    private static final Pattern SPACE = Pattern.compile(" ");
    public static void main(String[] args) {

        if (args.length < 1) {
            System.err.println("Usage: JavaWordCount <file>");
            System.exit(1);
        }

        SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount");
        SparkContext sparkContext = SparkContext.getOrCreate(sparkConf);

        JavaRDD<String> lines = sparkContext.textFile(args[0], 1).toJavaRDD();
        JavaRDD<String> words = lines.flatMap(line -> Arrays.asList(SPACE.split(line)).iterator());
        JavaPairRDD<String, Integer> ones = words.mapToPair(word -> new Tuple2<>(word, 1));
        JavaPairRDD<String, Integer> counts = ones.reduceByKey(Integer::sum);

        List<Tuple2<String, Integer>> output = counts.collect();
        for (Tuple2<?,?> tuple : output) {
            System.out.println(tuple._1() + ": " + tuple._2());
        }
        sparkContext.stop();
    }
}
```
3. 全局单利对象的静态方法

```java
```