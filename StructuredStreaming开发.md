#### Source
Structured Streaming中将输入源统一抽象为Source，这样做的好处有很多，最重要的一点就是可以保证在故障后可以使用检查点偏移来重放数据。接下来就看一下如何实现回放数据的。
```java
public interface Source extends BaseStreamingSource {
    StructType schema();

    Option<Offset> getOffset();

    Dataset<Row> getBatch(final Option<Offset> start, final Offset end);

    default void commit(final Offset end) {
    }

    void stop();
}
```
所有的Source都有getOffset方法用于获取偏移量，这样就可以在必要的时候进行数据回放了。
下面看一下自带的几个常用Source：
- **File source**

   用于读取本地获取分布式文件系统中指定目录下写入的文件，并且按照文件修改时间的顺序进行处理，可以通过latestFirst参数进行设置。每一批数据都会被赋予一个递增的编号，getOffset方法就是通过这个编号进行数据回放的。

- **Kafka source** 

   Kafka本身数据结构中就自带offset，可以很好地和接口中getOffset相匹配。

- **Socket source（测试）**

   读取Socke中的数据，无法进行数据回放

- **Rate source（测试）**

   以每秒指定的行数生成数据，每个输出行包含`timestamp`和`value`，`timestamp`是`Timestamp`类型的属性，表示事件发生时间，`value`是`Long`类型，从0开始递增的数据。
```java
SparkSession spark = SparkSession
                .builder()
                .appName("SourceApp")
                .getOrCreate();

StructType userSchema = new StructType()
    .add("name", "string")
    .add("age", "integer");

Dataset<Row> csvDF = spark
    .readStream()
    .option("sep", ";")
    .schema(userSchema)
    .csv("./csv");
logger.info("{}", csvDF.isStreaming());
csvDF.printSchema();
```

#### Sink

Sink是Structured Streaming对输出过程的统一抽象，也同样做到了对用户透明的容错处理。依然来看一下大致的实现原理：

```java
public interface Sink extends BaseStreamingSink {
    void addBatch(final long batchId, final Dataset<Row> data);
}
```

```java
// FileStreamSink
public void addBatch(final long batchId, final Dataset<Row> data) {
    if (batchId <= BoxesRunTime.unboxToLong(this.fileLog().getLatest().map((x$1) -> {
        return BoxesRunTime.boxToLong($anonfun$addBatch$1(x$1));
    })
    // ......    
}
```

以FileStreamSink为例，每次保存数据时都会根据批处理编号（batchId）判断是不是重复提交的数据。

自带的常用Sink：

- **Kafka Sink**

  保证处理-输出的这个环节做到完美的“恰好一次”，而只能做到“至少一次”，因此需要保障幂等输出或者使用``dropDuplicates``进行去重。

- **ForeachBatch Sink**

  `foreachBatch`在每个微批处理的输出上允许任意操作和自定义逻辑。默认情况下`foreachBatch`仅提供至少一次写入保证。但是，您可以使用提供给该函数的batchId作为对输出进行重复数据删除并获得一次准确保证的方式。

  以`foreachBatch`为例，两个参数微批处理的输出数据的数据帧或数据集和微批处理的唯一ID。

  ```java
  streamingDatasetOfString.writeStream().foreachBatch(
    new VoidFunction2<Dataset<String>, Long> {
      public void call(Dataset<String> dataset, Long batchId) {
        // Transform and write batchDF
      }    
    }
  ).start();
  ```

  通常遇到需要**重用现有的批处理数据源**和**写入多个位置**时会使用到`foreachBatch`。

- **Foreach Sink**

  可以使用来表示自定义编写器逻辑`foreach`。具体来说，可以通过将其划分为三种方法表示数据写入的逻辑：`open`，`process`，和`close`。

  ```java
  public abstract class ForeachWriter<T> implements Serializable {
      public abstract boolean open(final long partitionId, final long epochId);
  
      public abstract void process(final T value);
  
      public abstract void close(final Throwable errorOrNull);
  }
  ```

  这3个方法会在Executor上依次被调用，open方法的参数partitionId代表了输出分区ID, version是一个单调递增的ID，随着每次触发而增加，我们可以通过两个参数判断这一批数据是否继续输出。如果返回值为true，就调用process方法，如果返回值为false，则不会调用process方法。每当调用open方法时，close方法也会被调用（除非JVM因为某些错误而退出），因此我们可以在open方法中打开外部存储连接，并在close方法中关闭外部存储连接。

#### Triggers

流查询的触发器设置定义了流数据处理的时间，无论该查询是作为具有固定批处理间隔的微批查询还是作为连续处理查询执行。以下是受支持的各种触发器。

| **Trigger Type**               | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| *未指定（默认）*               | 如果未显式指定触发器设置，则默认情况下，查询将以微批处理模式执行，在该模式下，前一个微批处理完成后将立即生成微批处理。 |
| **固定间隔微批**               | 查询将以微批次模式执行，在该模式下，微批次将按用户指定的时间间隔启动。如果前一个微批处理在该间隔内完成，则引擎将等待该间隔结束，然后再开始下一个微批处理。 如果前一个微批处理花费的时间比间隔要长（例如，缺少间隔边界），则下一个微批处理将在前一个微批处理完成后立即开始（即，它不会等待下一个间隔边界） ）。 如果没有新数据可用，则不会启动微批量。 |
| **一次性微批量**               | 该查询将仅执行一个微批处理来处理所有可用数据，然后自行停止。在您要定期启动群集，处理自上一周期以来可用的所有内容然后关闭群集的情况下，这很有用。在某些情况下，这可能会节省大量成本。 |
| **以固定的检查点间隔连续进行** | 查询将以新的低延迟，连续处理模式执行                         |

```java
import org.apache.spark.sql.streaming.Trigger

// Default trigger (runs micro-batch as soon as it can)
df.writeStream
  .format("console")
  .start();

// ProcessingTime trigger with two-seconds micro-batch interval
df.writeStream
  .format("console")
  .trigger(Trigger.ProcessingTime("2 seconds"))
  .start();

// One-time trigger
df.writeStream
  .format("console")
  .trigger(Trigger.Once())
  .start();

// Continuous trigger with one-second checkpointing interval
df.writeStream
  .format("console")
  .trigger(Trigger.Continuous("1 second"))
  .start();
```

#### Window

```java
SparkSession session = SparkSession
                .builder()
                .appName("JavaStructuredNetworkWordCount")
                .getOrCreate();

session.sparkContext().setLogLevel(LogLevel.ERROR.getLabel());

// Create DataFrame representing the stream of input lines from connection to localhost:1999
Dataset<Row> lines = session.readStream()
    .format("socket")
    .option("host", "localhost")
    .option("port", 1999)
    .option("includeTimestamp", true)
    .load();

// Split the lines into words
Dataset<Row> words = lines
    .as(Encoders.tuple(Encoders.STRING(), Encoders.TIMESTAMP()))
    .flatMap(new FlatMapFunction<Tuple2<String, Timestamp>, Tuple2<String, Timestamp>>() {
        @Override
        public Iterator<Tuple2<String, Timestamp>> call(Tuple2<String, Timestamp> line) throws Exception {
            return Arrays.stream(line._1.split(" ")).map(word -> new Tuple2<>(word, line._2)).iterator();
        }
    }, Encoders.tuple(Encoders.STRING(), Encoders.TIMESTAMP()))
    .toDF("word", "timestamp");

// Generate running word count
Dataset<Row> wordCounts = words.groupBy(
    functions.window(
        col("timestamp"),
        "10 minutes",
        "5 minutes"
    ),
    col("word")
).count().orderBy("window");

// Start running the query that prints the running counts to the console
StreamingQuery query = wordCounts.writeStream()
    .format("console")
    .outputMode(OutputMode.Complete())
    .start();

query.awaitTermination();
```

上面的代码表示10分钟的窗口内对字数进行计数每5分钟更新一次，窗口跨度也是结果表的一个字段该字段名为window。

![spark006](http://git.nuozhilin.site/luzhong/images/raw/branch/master/spark008.png)

#### Watermarking

聚合的时间窗口是以数据的事件时间为准的，那么这样一定会存在晚到和乱序数据的问题。例如，应用程序可以在12:11接收在12:04（即事件时间）生成的单词。应用程序应使用12:04而不是12:11来更新窗口的旧计数`12:00 - 12:10`。结构化流可以长时间保持部分聚合的中间状态，以便后期数据可以正确更新旧窗口的聚合。但是要连续几天运行此查询，系统必须限制其累积的中间内存状态量。这意味着系统需要知道何时可以从内存中状态删除旧的聚合，因为应用程序将不再接收该聚合的最新数据，因此提供了**水位线**机制来告诉Spark何时丢弃迟到的数据。

```java
// Generate running word count
Dataset<Row> wordCounts = words
    .withWatermark("timestamp", "10 minutes")
    .groupBy(
    functions.window(
        col("timestamp"),
        "10 minutes",
        "5 minutes"
    ),
    col("word")
).count().orderBy("window");
```

上述代码表示：将“ 10分钟”定义为允许数据晚到的阈值。如果此查询在“更新输出”模式下运行，则引擎将在“结果表”中保持窗口的更新计数，直到该窗口早于水位线为止。

![spark006](http://git.nuozhilin.site/luzhong/images/raw/branch/master/spark009.png)

#### Join
##### Stream-static Joins
##### Stream-stream Joins
#### Unsupported Operations
#### Recovering