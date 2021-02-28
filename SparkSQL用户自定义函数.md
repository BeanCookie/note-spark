#### 用户自定义函数
Spark SQL虽然内置了许多函数，但是使用这些基础函数去处理复杂逻辑时依旧十分繁琐。因此Spark SQL提供了用户自定义函数(UDF)这一机制，可以让用户根据自己的需求随心所欲的定义各种函数并在SQL中使用。

#### 示例1：根据配置将时间戳转换成指定时区

##### 数据准备
click_log.csv
user_id：用户ID，click_article_id：点击文章ID，click_timestamp：时间戳
```csv
user_id,click_article_id,click_timestamp
249999,160974,1506959142820
249999,160417,1506959172820
249998,160974,1506959056066
249998,202557,1506959086066
249997,183665,1506959088613
249997,181686,1506959118613
249996,160974,1506959091575
249995,300470,1506959097865
249995,16129,1506960060854
249995,160974,1506960090854
```

time_zone.csv
user_id：用户ID，time_zone：用户所属时区
```csv
user_id,time_zone
249999,UTC+8
249998,UTC+7
249997,UTC+18
```

##### 在代码中定义UDF
```java
SparkSession sparkSession = SparkSession
                .builder()
                .appName("Spark SQL UDF example")
                .getOrCreate();
// 定义名为DateFormat的函数
sparkSession.udf().register("DateFormat", (UDF2<String, String, String>) (timestamp, timeZone) -> {
            if (Objects.isNull(timeZone)) {
                timeZone = "UTC+8";
            }
            // 将时间戳转换到指定时区
            return Instant.ofEpochMilli(Long.parseLong(timestamp))
                    .atZone(ZoneId.of(timeZone))
                    .format(DATE_TIME_FORMATTER);

        }, DataTypes.StringType);
```

##### 在Spark SQL中使用UDF
```java


sparkSession.read()
                .option("header", true)
                .csv("./click_log.csv")
                .createOrReplaceTempView("click_log");

        sparkSession.read()
                .option("header", true)
                .csv("./time_zone.csv")
                .createOrReplaceTempView("time_zone");

sparkSession.sql("SELECT \n" +
    "a.user_id, \n" +
    "a.click_article_id, \n" +
    "a.click_timestamp, \n" +
    "b.time_zone, \n" +
    "DateFormat(a.click_timestamp, b.time_zone) \n" +
    "FROM \n" +
    "click_log as a\n" +
    "LEFT JOIN time_zone as b ON a.user_id = b.user_id")
    .show(100);
```

##### SQL结果
这样就实现了根据用户配置将时间戳转换到指定时区了
```shell
+-------+----------------+---------------+---------+------------------------------------------+
|user_id|click_article_id|click_timestamp|time_zone|UDF:DateFormat(click_timestamp, time_zone)|
+-------+----------------+---------------+---------+------------------------------------------+
| 249999|          160974|  1506959142820|    UTC+8|                       2017-10-02 23:45:42|
| 249999|          160417|  1506959172820|    UTC+8|                       2017-10-02 23:46:12|
| 249998|          160974|  1506959056066|    UTC+7|                       2017-10-02 22:44:16|
| 249998|          202557|  1506959086066|    UTC+7|                       2017-10-02 22:44:46|
| 249997|          183665|  1506959088613|   UTC+18|                       2017-10-03 09:44:48|
| 249997|          181686|  1506959118613|   UTC+18|                       2017-10-03 09:45:18|
| 249996|          160974|  1506959091575|     null|                       2017-10-02 23:44:51|
| 249995|          300470|  1506959097865|     null|                       2017-10-02 23:44:57|
| 249995|           16129|  1506960060854|     null|                       2017-10-03 00:01:00|
| 249995|          160974|  1506960090854|     null|                       2017-10-03 00:01:30|
+-------+----------------+---------------+---------+------------------------------------------+
```