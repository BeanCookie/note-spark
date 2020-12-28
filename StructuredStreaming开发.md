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

Sink是Structured Streaming对输出过程的统一抽象，也同样做到了对容错

#### Output Modes
#### Triggers
#### Window
#### Watermarking
#### Join
##### Stream-static Joins
##### Stream-stream Joins
#### Unsupported Operations
#### Recovering