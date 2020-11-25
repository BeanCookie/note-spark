#### Maven配置
```xml
<dependency>
    <groupId>org.apache.spark</groupId>
    <artifactId>spark-core_2.12</artifactId>
    <version>2.4.7</version>
</dependency>
```
#### 创建RDD
##### 获取执行环境
##### Java
```java
SparkConf sparkConf = new SparkConf().setMaster("local[*]").setAppName("WordCount");
SparkContext sparkContext = SparkContext.getOrCreate(sparkConf);
```

##### SparkShell
系统会自动创建一个名为sc的SparkContext对象
```shell
sc
# res0: org.apache.spark.SparkContext = org.apache.spark.SparkContext@6e3acd7
```
##### 从集合中获取RDD
```shell
val data = Array(1, 2, 3, 4, 5)
# data: Array[Int] = Array(1, 2, 3, 4, 5)
val distData = sc.parallelize(data)
# distData: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:26
```
##### 从外部文件中获取RDD
```shell
val distFile = sc.textFile("/usr/local/spark-local/NOTICE")
# distFile: org.apache.spark.rdd.RDD[String] = /usr/local/spark-local/NOTICE MapPartitionsRDD[3] at textFile at <console>:24
```
#### 转化算子

#### Value类型

#### ``map(func)``
- 含义: 返回一个新的RDD，该数据集是通过将源的每个元素传递给函数func形成的
- 示例: 计算每个元素的平方
```shell
distData.collect()
# Array[Int] = Array(1, 2, 3, 4, 5)

val mapDistData = distData.map(v => v * v)
mapDistData.collect()
# Array[Int] = Array(1, 4, 9, 16, 25)
```

#### ``filter(func)``
- 含义: 返回一个新的RDD，该数据集是通过选择源中func返回true的那些元素形成的
- 示例: 筛选大于3的数据
```shell
distData.collect()
# Array[Int] = Array(1, 2, 3, 4, 5)

val filterDistData = distData.filter(v => v > 3)
filterDistData.collect()
# Array[Int] = Array(4, 5)
```

#### ``flatMap(func)``
- 含义: 与map相似但是每个输入项都可以映射到0个或多个输出项（因此func应该返回数组而不是单个对象）
- 示例:
```shell
val stringData = Array("1", "12", "123", "1234", "12345")
val stringDistData = sc.parallelize(stringData)
stringDistData.collect()
# Array[String] = Array(1, 12, 123, 1234, 12345)
val flatMapDistData = stringDistData.flatMap(v => v.split(""))
flatMapDistData.collect()
# Array[String] = Array(1, 1, 2, 1, 2, 3, 1, 2, 3, 4, 1, 2, 3, 4, 5)
```

#### ``mapPartitions(func)``
- 含义: 与map相似但是会接受一个分区中的所有数据，而map只会接收单个元素
- 示例:
```shell

```

#### ``mapPartitionsWithIndex(func)``
- 含义: 与mapPartitions类似而且可以获取分区索引的整数值
- 示例:
```shell

```

#### ``sample(withReplacement, fraction, seed)``
- 含义: 
- 示例:
```shell

```

#### ``distinct([numPartitions]))``
- 含义: 返回一个新的RDD其中包含源RDD去重后的结果
- 示例:
```shell

```

#### ``sortBy(func,[ascending], [numTasks])``
- 含义: 使用func先对数据进行处理，按照处理后的数据比较结果排序默认为正序
- 示例:
```shell

```

#### ``pipe(command, [envVars])``
- 含义: 通过SHELL命令（例如Perl或bash脚本）通过管道传输RDD的每个分区。将RDD元素写入进程的stdin，并将输出到其stdout的返回行作为String类型的RDD返回
- 示例:
```shell

```

#### ``coalesce(numPartitions)``
- 含义: 缩减分区数
- 示例:
```shell

```

#### ``repartition(numPartitions)``
- 含义: 根据分区数重新通过网络随机洗牌所有数据
- 示例:
```shell

```

---

#### Dubbo Value类型
#### ``union(otherDataset)``
- 含义: 返回一个新的RDD，其中包含源RDD和输入RDD两者元素的并集
- 示例:
```shell

```

#### ``intersection(otherDataset)``
- 含义: 返回一个新的RDD，其中包含源RDD和输入RDD两者元素的交集
- 示例:
```shell

```

#### ``subtract (otherDataset)``
- 含义: 返回一个新的RDD，其中包含源RDD和输入RDD两者元素的差集
- 示例:
```shell

```

#### ``cartesian(otherDataset)``
- 含义: 笛卡尔积
- 示例:
```shell

```

#### ``zip(otherDataset)``
- 含义: 将两个RDD组合成(K,V)形式的RDD,两者元素个数必须相等否则会抛出异常
- 示例:
```shell

```

---

#### KV类型

#### ``sortByKey([ascending], [numPartitions])``
- 含义: 
- 示例:
```shell

```

#### ``groupByKey([numPartitions])``
- 含义: 
- 示例:
```shell

```

#### ``reduceByKey(func, [numPartitions])``
- 含义: 
- 示例:
```shell

```

#### ``foldByKey``
- 含义: 
- 示例:
```shell

```

#### ``combineByKey``
- 含义: 
- 示例:
```shell

```

#### ``mapValues``
- 含义: 
- 示例:
```shell

```

#### ``aggregateByKey(zeroValue)(seqOp, combOp, [numPartitions])``
- 含义: 
- 示例:
```shell

```

#### ``partitionBy``
- 含义: 
- 示例:
```shell

```

#### ``repartitionAndSortWithinPartitions(partitioner)``
- 含义: 根据给定的分区程序对RDD进行重新分区，并在每个结果分区中，按其键对记录进行排序。这比repartition在每个分区内调用然后排序更为有效，因为它可以将排序推入洗牌机制
- 示例:
```shell

```

#### ``join(otherDataset, [numPartitions])``
- 含义: 
- 示例:
```shell

```

#### ``cogroup(otherDataset, [numPartitions])``
- 含义: 在类型为(K,V)和(K,W)的 RDD 上调用，返回一个(K,(Iterable<V>,Iterable<W>))类 型的 RDD
- 示例:
```shell

```

#### 行动算子

#### RDD分区