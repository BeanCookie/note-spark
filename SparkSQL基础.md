#### Spark SQL起源
Hive是建立在Hadoop之上的数据仓库基础框架，也是最早运行在Hadoop上的SQL on Hadoop工具之一，但是Hive是基于MapReduce的计算过程有大量的中间磁盘落地过程，消耗了大量的IO，这大大降低了运行效率。

Shark直接建立在Apache Hive上，为了实现Hive兼容，在HQL方面重用了Hive中HQL的解析、逻辑执行计划翻译、执行计划优化等逻辑，可以近似认为仅将物理执行计划从MR作业替换成了Spark作业。

Spark SQL从HQL被解析成抽象语法树（AST）起就全部接管了。执行计划生成和优化都由Catalyst负责。借助Scala的模式匹配等函数式语言特性，利用Catalyst开发执行计划优化策略比Hive要简洁得多。

#### DataFrame
DataFrame的概念并不是Spark独创的，在数据科学领域中，DataFrame抽象了矩阵，如R、pandas中的DataFrame，在数据工程领域，在Spark SQL我们可以把DataFrame的结构想象成关系型数据库中的一张表。它是Spark SQL与外部数据源之间的桥梁。与RDD最大的不同在于，RDD仅仅是一条条数据的集合，并不了解每一条数据的内容是怎样的，而DataFrame不仅提供了更加丰富的算子操作，还清楚地知道该数据集包含哪些列，每一列数据的名称、类型

#### 创建DataFrame

##### 获取SparkSession
SparkSession对象不仅为用户提供了创建DataFrame对象、读取外部数据源并转化为DataFrame对象以及执行sql查询的API，还负责记录着用户希望Spark应用如何在Spark集群运行的控制、调优参数，是Spark SQL的上下文环境。
```java
SparkSession sparkSession = SparkSession
                .builder()
                .appName("Spark SQL basic example")
                .getOrCreate();
```

通过SparkSession我们可以从现有RDD、Hive表和外部数据源中创建DataFrame。

```java
Dataset<Row> dataset = sparkSession.read()
                .option("header", true)
                .csv("/vagrant_data/pokemon0820.csv");
```

#### 将RDD转化为DataFrame

##### 使用反射机制推理出Schema
支持将JavaBean的RDD自动转换为DataFram<T>，当前Spark SQL不支持包含Map字段的JavaBean，不过支持嵌套JavaBean和List或Array字段，可以通过创建一个实现Serializable并具有其所有字段的getter和setter方法的类来创建JavaBean。
```java
JavaRDD<Pokemon> pokemonRDD = dataset.javaRDD()
                .map(row -> {
                    Pokemon pokemon = new Pokemon();
                    pokemon.setAbilities(row.getString(0));
                    ...
                    return pokemon;
                });
sparkSession.createDataFrame(pokemonRDD, Pokemon.class);
```
##### 由开发者指定Schema
当无法提前定义JavaBean类时（例如，记录的结构编码为字符串，或者将解析文本数据集并且为不同的用户设计不同的字段）
```java
String schemaString = "abilities,against_bug,against_dark,...";
List<StructField> fields = new ArrayList<>();
for (String fieldName : schemaString.split(",")) {
    StructField field = DataTypes.createStructField(fieldName, DataTypes.StringType, true);
    fields.add(field);
}
StructType schema = DataTypes.createStructType(fields);
JavaRDD<Row> pokemonRDD = dataset.javaRDD()
                .map(row -> RowFactory.create(row.getString(0), row.getString(1,...));
sparkSession.createDataFrame(pokemonRDD, schema);
```

- func printSchema
- 输出以树格式输出DataFrame对象的Schema信息
```java
dataset.printSchema();
```
```
root
 |-- abilities: string (nullable = true)
 |-- against_bug: string (nullable = true)
 |-- against_dark: string (nullable = true)
```

#### 算子风格
- select
```java
dataset.select("name", "percentage_male").show(3);
```
```
+---------+---------------+
|     name|percentage_male|
+---------+---------------+
|Bulbasaur|           88.1|
|  Ivysaur|           88.1|
| Venusaur|           88.1|
+---------+---------------+
```
- where
```java
dataset.select("name", "percentage_male")
                .where(col("percentage_male").lt(50))
                .show(3);
```
```
+---------+---------------+
|     name|percentage_male|
+---------+---------------+
| Nidoran♀|              0|
| Nidorina|              0|
|Nidoqueen|              0|
+---------+---------------+
```
- orderBy
```java
dataset.select("name", "weight_kg")
                .orderBy(col("weight_kg").desc())
                .show(3);
```
```
+----------+---------+
|      name|weight_kg|
+----------+---------+
|   Cosmoem|    999.9|
|Celesteela|    999.9|
|  Bergmite|     99.5|
+----------+---------+
```
- groupBy
```java
dataset.groupBy("type1").count()
                .orderBy(col("count").desc())
                .show();
```
```
+--------+-----+
|   type1|count|
+--------+-----+
|   water|  114|
|  normal|  105|
|   grass|   78|
|     bug|   72|
| psychic|   53|
|    fire|   52|
|    rock|   45|
|electric|   39|
|  poison|   32|
|  ground|   32|
|    dark|   29|
|fighting|   28|
|   ghost|   27|
|  dragon|   27|
|   steel|   24|
|     ice|   23|
|   fairy|   18|
|  flying|    3|
+--------+-----+
```

#### SQL风格
##### 零时表
在Spark SQL模块上直接执行sql语句的查询需要首先将标志着结构化数据源的DataFrame对象注册成临时表，Spark SQL的SQL接口全面支持SQL的select标准语法，包括SELECT DISTINCT、FROM子句、WHERE子句、ORDER BY字句、GROUP BY子句、HAVING子句、JOIN子句，还有典型的SQL函数，例如AVG()、COUNT()、MAX()、MIN()等

##### 全局临时表
全局临时表（global temporary view）于临时表（temporaryview）是相对的，全局临时表的作用范围是某个Spark应用程序内所有会话（SparkSession），它会持续存在，在所有会话中共享，直到该Spark应用程序终止

```java
dataset.createOrReplaceTempView("pokemon");
sparkSession.sql("SELECT type1, COUNT(type1) FROM pokemon GROUP BY type1 ORDER BY count(type1) DESC").show();
```
```
+--------+-----+
|   type1|count|
+--------+-----+
|   water|  114|
|  normal|  105|
|   grass|   78|
|     bug|   72|
| psychic|   53|
|    fire|   52|
|    rock|   45|
|electric|   39|
|  poison|   32|
|  ground|   32|
|    dark|   29|
|fighting|   28|
|   ghost|   27|
|  dragon|   27|
|   steel|   24|
|     ice|   23|
|   fairy|   18|
|  flying|    3|
+--------+-----+
```

##### 窗口函数

##### 用户自定义函数

#### Dataset
DataFrame本质上正是特殊的Dataset：``DataFrame == Dataset<Row>``

#### 创建Dataset
Dataset[T]中对象的序列化并不使用Java标准序列化或Kryo，而是使用专门的编码器对对象进行序列化以便通过网络进行处理或传输。虽然编码器和标准序列化都负责将对象转换为字节，但编码器是根据Dataset[T]的元素类型（T）动态生成，并且允许Spark无须将字节反序列化回对象的情况下即可执行许多操作（如过滤、排序和散列），因此避免了不必要的反序列化导致的资源浪费更加高效

```java
public static class Pokemon {
    private String abilities;
    private String against_bug;
    private String against_dark;
    // ...
    // get and set method
```
```java
Dataset<Pokemon> pokemonDataset = sparkSession.read().csv("/vagrant_data/pokemon0820.csv").as(Encoders.bean(Pokemon.class));
        pokemonDataset.printSchema();
```

#### SparkSQL
##### 查询语句
- SELECT与FROM
- WHERE
- GROUP BY
- HAVING
- UNION
- ORDER BY
- LIMIT
- JOIN
##### 内置函数
- cast(value AS type) → type
- log(double base, Column a)
- exp(Column e)
- factorial(Column e)
- split(Column str, String pattern)
- substring(Column str, int pos, int len)
- concat(Column... exprs)
- translate(Column src, String matchingString, StringreplaceString)
- bin(Column e)
- hex(Column e)
- base64(Column e)
- unbase64(Column e)
- current_date()
- current_timestamp()
- date_format(Column dateExpr, String format)
- regexp_extract(Column e, String exp, int groupIdx)
- regexp_replace(Column e, String pattern, Stringreplacement)
- to_json(Column e)
- from_json(Column e, Column Schema)
- get_json_object(Column e, String path)
- parse_url(string urlString, string partToExtract [,stringkeyToExtract])
- countDistinct(Column expr, Column... exprs)
- sumDistinct(Column e)
- approxCountDistinct(String columnName, doublersd)
- avg(Column e)
- count(Column e)
- first(Column e, [Boolean ignoreNulls])
- last(Column e, [Boolean ignoreNulls])
- max(Column e)
- min(Column e)
- sum(Column e)
- skewness(Column e)
- stddev_samp(Column e)
- stddev(Column e)
- stddev_pop(Column e)
- var_samp(Column e)
- variance(Column e)
- var_pop(Column e)
##### 窗口函数
##### 自定义函数