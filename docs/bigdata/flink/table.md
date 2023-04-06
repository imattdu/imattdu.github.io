## 概述





![](https://raw.githubusercontent.com/imattdu/img/main/img/202304070107197.png)







ProcessFunction -> DataStream -> Table -> SQL

ProcessFunction:  事件的时间信息、注册定时器、自定义状态，进行有状态的流处理



不过在企业实际应用中，往往会面对大量类似的处理逻辑，所以一般会将底层 API 包装 成更加具体的应用级接口，如直接是使用SQL



## quickstart





### 引入依赖



``` xml
        <!--flinkTableAPI使用, tableAPI 到DataStream连接支持-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java-bridge_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <!--如需本地ide运行，还须引入以下依赖-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner-blink_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-scala_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
```



这里主要添加的依赖是一个“计划器”(planner)，它是 Table API 的核心组件，负责提供运行时环境，并生成程序的执行计划。这里我们用到的是新版的blink planner。由于Flink安装包的 lib 目录下会自带 planner，所以在生产集群环境中提交的作业不需要打包这个依赖。





如果想实现自定义的数据格式来做序列化，可以引入下面的依赖

```
<dependency>
   <groupId>org.apache.flink</groupId>
   <artifactId>flink-table-common</artifactId>
   <version>${flink.version}</version>
</dependency>
```



### code



```java
package com.matt.apitest.table;

import com.matt.apitest.beans.Event;
import com.matt.apitest.source.SourceTest4_UDF;
import org.apache.flink.api.common.eventtime.SerializableTimestampAssigner;
import org.apache.flink.api.common.eventtime.WatermarkStrategy;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.SingleOutputStreamOperator;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.flink.types.Row;

import java.time.Duration;

import static org.apache.flink.table.api.Expressions.$;

/**
 * @author matt
 * @create 2023-03-24 00:10
 * @desc xxx
 */
public class SimpleTableCase {

    public static void main(String[] args) throws Exception {
        // 1.create env
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        // 2.build sourceStream
        SingleOutputStreamOperator<Event> eventStream = env.addSource(new SourceTest4_UDF.ParallelCustomSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));

        // 3.创建表执行环境
        StreamTableEnvironment streamTableEnv = StreamTableEnvironment.create(env);

        // 4. createTable
        Table eTable = streamTableEnv.fromDataStream(eventStream);
        Table resT = streamTableEnv.sqlQuery("select user, url, `timestamp` from " + eTable);

        Table resT2 = resT.select($("user"), $("url"))
                .where($("user").isEqual("matt"));

        // 5.table out
        DataStream<Row> rowDataStream = streamTableEnv.toDataStream(resT);
        DataStream<Row> res2 = streamTableEnv.toDataStream(resT2);
        rowDataStream.print("res1");
        res2.print("res2");
        env.execute();

        // 更新流 使用
        //streamTableEnv.toChangelogStream("");
    }
}
```

## commonAPI



### 程序架构

DataStream: 读取数据源Source -> 转换Transform -> 输出数据Sink



Table: 创建表环境，设置输入输出表 -> 执行SQL对表进行查询转换 -> 写入到输出表



### 创建表环境



表环境：运行的环境



功能：

- 注册 Catalog 和表;
- 执行 SQL 查询;
- 注册用户自定义函数(UDF);
- DataStream 和表之间的转换。



Catalog: 主要用来管理所有数据库(database)和表(table)的元数据(metadata)。默认的 Catalog 就叫作 default_catalog。





code



**当前表环境的执行模式和计划器(planner)。执行模式有批处理和流处理两种选择，默认是流处理模式;计划器默认使 用 blink planner。**

```java
				// 常用
        //StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //env.setParallelism(1);
        //StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // 1 定义环境配置 创建执行环境
        EnvironmentSettings settings = EnvironmentSettings.newInstance()
                .inStreamingMode()
                .useBlinkPlanner()
                .build();
        TableEnvironment tableEnv = TableEnvironment.create(settings);

        /*// 老版本 流处理
        EnvironmentSettings settings1 = EnvironmentSettings.newInstance()
                .inStreamingMode()
                .useOldPlanner()
                .build();
        TableEnvironment tableEnv1 = TableEnvironment.create(settings1);

        // 老版本 批处理
        ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment();
        BatchTableEnvironment batchTableEnvironment = BatchTableEnvironment.create(batchEnv);
				*/
```



### 创建表

#### 概述

表在环境中有一个唯一的 ID，由三部分组成:目录(catalog)名，数据库(database)名，以及表名。在默认情况下，目录名为 default_catalog，数据库名为default_database。所以如果我们直接创建一个叫作 MyTable 的表，它的 ID 就是: **default_catalog.default_database.MyTable**



创建表的方式有俩种 连接器connector 和虚拟表virtual tables



设置cataLog&database

``` java
tEnv.useCatalog("custom_catalog");
tEnv.useDatabase("custom_database");
```





#### 连接器



定义一个ddl 语句 ，然后执行



```java
 				// 创建表
        String createDDL = "CREATE TABLE clickTable (" +
                "`user` STRING, " +
                "ts BIGINT, " +
                "v STRING" +
                ") WITH (" +
                "'connector' = 'filesystem'," +
                "'path' = '/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt'," +
                "'format' = 'csv'" +
                ")"; // , 分割的文本文件
        tableEnv.executeSql(createDDL);

        String createOutDDL = "CREATE TABLE outTable (" +
                "`user` STRING, " +
                "ts BIGINT " +
                ") WITH (" +
                "'connector' = 'filesystem'," +
                "'path' = 'out'," +
                "'format' = 'csv')"; // , 分割的文本文件
        tableEnv.executeSql(createOutDDL);
```





#### 虚拟表

查询后得到Table -> 注册虚拟表

``` java
				Table clickTable = tableEnv.from("clickTable");
        // 这里不用写反引号
        Table mattTable = clickTable.where($("user").isEqual("matt"));

        tableEnv.createTemporaryView("mattTable",mattTable);
```



### 表的查询

支持SQL查询和TableAPI



#### SQL

``` JAVA
String aggResSQL = "SELECT user, count(ts) from clickTable group by user";
String aggResDDL = "CREATE TABLE printOutTable (" +
                "`user` STRING, " +
                "cnt BIGINT " +
                ") WITH (" +
                "'connector' = 'print')";
Table aggResTable = tableEnv.sqlQuery(aggResSQL);
```



- SELECT语句查询，调用sqlQuery
- INSERT语句写入到outTable,调用executeSQL

#### TableAPI

``` 
Table maryClickTable = eventTable
       .where($("user").isEqual("Alice"))
       .select($("url"), $("user"));
```

$指定表的一个字段





#### 结合使用

```
Table clickTable = tableEnvironment.sqlQuery("select url, user from " +
eventTable);
```

也可以将eventTable 注册到虚拟表就可以直接使用






-u 删除

+u 更新



```sh
3> +I[matt, 1]
2> +I[sensor_7, 1]
10> +I[sensor_1, 1]
5> +I[sensor_6, 1]
5> -U[sensor_6, 1]
3> -U[matt, 1]
5> +U[sensor_6, 2]
3> +U[matt, 2]
5> -U[sensor_6, 2]
5> +U[sensor_6, 3]
```





sink 不支持更新操作







累计窗口只会计算一次 没有5-15



滑动窗口会计算到不同窗口中。有5-15





1. 累计窗口：

累计窗口是一个基于时间的固定大小的窗口，它包含从流开始时间到当前时间之间的所有事件。当一个事件进入窗口时，它会被保留在窗口中，直到窗口关闭并进行聚合操作。累计窗口通常用于计算全局的聚合函数，例如对整个数据集进行求和、计数、平均值等操作。

1. 滚动窗口：

滚动窗口是一个基于时间的固定大小的窗口，它随着时间不断向前移动，并始终包含最近一段时间内的事件。当一个新的窗口开始时，之前的窗口将被关闭并进行聚合操作。滚动窗口通常用于计算实时的统计信息，例如最近10秒钟内的平均值、最大值等。

因此，累计窗口和滚动窗口的最大区别在于它们对时间窗口的定义方式和计算方式。累计窗口是对整个数据集进行操作，而滚动窗口只对最近一段时间内的数据进行操作，可以实现实时计算和持续聚合。









``` 
./bin/sql-client.sh
```

