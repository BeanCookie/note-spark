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

```
##### 从外部文件中获取RDD

#### 转化算子
##### Value类型
##### KV类型

#### 行动算子

#### RDD分区