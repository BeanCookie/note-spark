#### Maven配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.zeaho</groupId>
    <artifactId>spark</artifactId>
    <version>1.0-SNAPSHOT</version>
    <repositories>
        <repository>
            <id>aliyun</id>
            <url>http://maven.aliyun.com/nexus/content/groups/public</url>
        </repository>
        <repository>
            <id>hortonworks</id>
            <url>https://repo.hortonworks.com/content/repositories/releases/</url>
        </repository>
    </repositories>

    <properties>
        <maven.compiler.target>1.8</maven.compiler.target>
        <maven.compiler.source>1.8</maven.compiler.source>
        <spark.version>2.4.7</spark.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-sql_2.12</artifactId>
            <version>${spark.version}</version>
        </dependency>
        <dependency>
            <groupId>com.hortonworks.shc</groupId>
            <artifactId>shc-core</artifactId>
            <version>1.1.0.3.1.6.7-4</version>
        </dependency>
    </dependencies>
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <createDependencyReducedPom>false</createDependencyReducedPom>
                            <filters>
                                <filter>
                                    <artifact>*:*</artifact>
                                    <excludes>
                                        <exclude>META-INF/*.SF</exclude>
                                        <exclude>META-INF/*.DSA</exclude>
                                        <exclude>META-INF/*.RSA</exclude>
                                    </excludes>
                                </filter>
                            </filters>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

Spark内置的读取数据源还是使用了TableInputFormat来读取HBase中的数据。这个TableInputFormat有一些缺点：

1. 一个 Task 里面只能启动一个 Scan 去 HBase 中读取数据；
2. TableInputFormat 中不支持 BulkGet；
3. 不能享受到Spark SQL内置的catalyst引擎的优化。
基于这些问题，来自Hortonworks的工程师们为我们带来了全新的Apache Spark—Apache HBase Connector，下面简称SHC。通过这个类库，我们可以直接使用Spark SQL将DataFrame中的数据写入到HBase中；而且我们也可以使用Spark SQL去查询HBase中的数据，在查询HBase的时候充分利用了catalyst引擎做了许多优化，比如分区修剪（partition pruning），列修剪（column pruning），谓词下推（predicate pushdown）和数据本地性（data locality）等等。因为有了这些优化，通过Spark查询HBase的速度有了很大的提升

定义HBase catalog：
1. 定义名为 pokemon 的 HBase 表的目录架构。
2. 将rowkey标识为abilities，并将Spark中使用的列名映射到HBase中使用的列族、列名和列类型。
```java
package com.zeaho.spark.constant;

/**
 * @author LuZhong
 */
public class CatalogConstants {
    public static final String POKEMON = "{\n" +
            "  \"table\": {\n" +
            "    \"namespace\": \"default\",\n" +
            "    \"name\": \"pokemon\"\n" +
            "  },\n" +
            "  \"rowkey\": \"abilities\",\n" +
            "  \"columns\": {\n" +
            "    \"abilities\": {\n" +
            "      \"cf\": \"rowkey\",\n" +
            "      \"col\": \"abilities\",\n" +
            "      \"type\": \"string\"\n" +
            "    },\n" +
            "    \"against_bug\": {\n" +
            "      \"cf\": \"cf1\",\n" +
            "      \"col\": \"against_bug\",\n" +
            "      \"type\": \"string\"\n" +
            "    },\n" +
            "    \"against_dark\": {\n" +
            "      \"cf\": \"cf2\",\n" +
            "      \"col\": \"against_dark\",\n" +
            "      \"type\": \"string\"\n" +
            "    },\n" +
            "  }\n" +
            "}\n" +
            "\n";
}
```

读取CSV再写入HBASE
```java
package com.zeaho.spark.sql;

import com.zeaho.spark.constant.CatalogConstants;
import org.apache.spark.sql.AnalysisException;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.execution.datasources.hbase.HBaseTableCatalog;

/**
 * @author LuZhong
 */
public class PokemonWriterApp {
    public static void main(String[] args) throws AnalysisException {
        SparkSession sparkSession = SparkSession
                .builder()
                .appName("Spark SQL PokemonWriterApp")
                .getOrCreate();

        Dataset<Row> dataset = sparkSession.read()
                .option("header", true)
                .csv("/vagrant_data/pokemon0820.csv");

        dataset.write()
                .option(HBaseTableCatalog.tableCatalog(), CatalogConstants.POKEMON)
                .option(HBaseTableCatalog.newTable(), "5")
                .format("org.apache.spark.sql.execution.datasources.hbase")
                .save();
        dataset.show(3);
    }
}
```

读取HBASE中的数据并用Spark SQL进行数据分析
```java
package com.zeaho.spark.sql;

import com.zeaho.spark.constant.CatalogConstants;
import org.apache.spark.sql.AnalysisException;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.execution.datasources.hbase.HBaseTableCatalog;

import static org.apache.spark.sql.functions.col;

/**
 * @author LuZhong
 */
public class PokemonReaderApp {
    public static void main(String[] args) throws AnalysisException {
        SparkSession sparkSession = SparkSession
                .builder()
                .appName("Spark SQL PokemonReaderApp")
                .getOrCreate();

        Dataset<Row> pokemonDataset = sparkSession.read()
                .option(HBaseTableCatalog.tableCatalog(), CatalogConstants.POKEMON)
                .format("org.apache.spark.sql.execution.datasources.hbase")
                .load();
        pokemonDataset.groupBy("type1").count()
                .orderBy(col("count").desc())
                .show();;
    }
}
```