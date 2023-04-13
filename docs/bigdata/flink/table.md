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

```xml
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





### 输出表

- 创建输出表
- 查询结果写入到输出表



``` java
 String createOutDDL = "CREATE TABLE outTable (" +
                "`user` STRING, " +
                "ts BIGINT " +
                ") WITH (" +
                "'connector' = 'filesystem'," +
                "'path' = 'out'," +
                "'format' = 'csv')"; // , 分割的文本文件
        tableEnv.executeSql(createOutDDL);

Table mattTable2 = tableEnv.sqlQuery("select user, ts from mattTable");

mattTable2.executeInsert("outTable");

```



### 表和流的转换

#### 表转换流 Table2DataStream



toDataStream

```java
tableEnv.toDataStream(aliceVisitTable).print();
```



toChangelogStream: 表的内容会发生更新

``` java
tableEnv.toChangelogStream(urlCountTable).print();
```

#### 流转换表

fromDataStream

```java
Table eventTable2 = tableEnv.fromDataStream(eventStream, $("timestamp"), 
$("url"));

// 将 timestamp 字段重命名为 ts
Table eventTable2 = tableEnv.fromDataStream(eventStream, $("timestamp").as("ts"), 
$("url"));
```



createTemporaryView

``` java
tableEnv.createTemporaryView("EventTable", eventStream, 
$("timestamp").as("ts"),$("url"));
```



fromChangelogStream

表环境还提供了一个方法 fromChangelogStream()，可以将一个更新日志流转换成表。这 个方法要求流中的数据类型只能是 Row，而且每一个数据都需要指定当前行的更新类型 （RowKind）；





#### 支持的数据类型



原子类型

int,double,string 和通用的数据类型（不可在拆分的数据类型）



Tuple类型

如果不做重命名，那么名字就是f0,f1,f2



pojo类型

如果不做重命名，字段名就是原来类中字段名称



Row

Row 类型还附加了一个属性 RowKind，用来表示当前行在更新操作中的类型。这样， Row 就可以用来表示更新日志流（changelog stream）中的数据

``` java
DataStream<Row> dataStream =
 env.fromElements(
 Row.ofKind(RowKind.INSERT, "Alice", 12),
 Row.ofKind(RowKind.INSERT, "Bob", 5),
 Row.ofKind(RowKind.UPDATE_BEFORE, "Alice", 12),
 Row.ofKind(RowKind.UPDATE_AFTER, "Alice", 100));
// 将更新日志流转换为表
Table table = tableEnv.fromChangelogStream(dataStream);
```





### case 

统计每个用户点击次数



``` java
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
       

        Table userClickCnt = streamTableEnv.sqlQuery("select user, count(url)  from " + eTable + " group by user");

        // 5.table out
        DataStream<Row> userClickCntStream = streamTableEnv.toChangelogStream(userClickCnt);
        userClickCntStream.print();
        env.execute();

    }
}

```





## 流处理中的表



### 动态表和持续查询



#### 动态表

如果我们保存了表在某一时刻的快照（snapshot），那么 接下来只要读取更新日志流，就可以得到表之后的变化过程和最终结果了



#### 持续查询

由于数据在不断变化，因此基于它定义 的 SQL 查询也不可能执行一次就得到最终结果。

随着新数据的到来而继续执行



![](https://raw.githubusercontent.com/imattdu/img/main/img/202304081720302.png)



### 流转换为动态表



无更新则使用 fromDataStream

有更新则使用 fromChangelogStream



几种流insert

仅插入流

- 新增insert：add 消息



撤回流retract

- 新增insert：add 消息
- 删除delete:retract消息
- 更新update：retract,add 更新后的消息



更新插入流upsert

- 更新插入：有一个唯一的key, 存在则更新不存在则插入
- 删除：删除消息



在表流转换是没有唯一的key 所以当前使用的是撤回流



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





## 时间属性和窗口



### 事件时间

1s 的延迟



函数说明

```txt
TO_TIMESTAMP
// 带有时区
TO_TIMESTAMP_LTZ



timestamp = "2022-04-12 12:00:00"
TO_TIMESTAMP(timestamp, 'yyyy-MM-dd HH:mm:ss')

timestamp = 1649826000L // 秒级时间戳
FROM_UNIXTIME(timestamp, 'yyyy-MM-dd HH:mm:ss')
```



事件时间是ms 级别时间戳



``` java
 String createDDL = "CREATE TABLE clickTable (" +
                "`user` STRING, " +
                "ts BIGINT, " +
                "url STRING," +
                "et AS TO_TIMESTAMP(FROM_UNIXTIME(ts))," +
                "WATERMARK FOR et as et - INTERVAL '1' SECOND" +
                ") WITH (" +
                "'connector' = 'filesystem'," +
                "'path' = '/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt'," +
                "'format' = 'csv'" +
                ")"; // , 分割的文本文件
        tableEnv.executeSql(createDDL);

        // 2.流转换成 指定
        SingleOutputStreamOperator<Event> clickStream = env.addSource(new SourceTest4_UDF.ParallelCustomSource())
                .assignTimestampsAndWatermarks(WatermarkStrategy.<Event>forBoundedOutOfOrderness(Duration.ZERO)
                        .withTimestampAssigner(new SerializableTimestampAssigner<Event>() {
                            @Override
                            public long extractTimestamp(Event event, long l) {
                                return event.timestamp;
                            }
                        }));

        Table clickTable = tableEnv.fromDataStream(clickStream, $("user"), $("url"), $("timestamp").as("ts"), $("et").rowtime());
        clickTable.printSchema();
```





### 处理时间



创建表时指定

ROCTIME()函数来指定当前的处理时间属性，返回的类型是 TIMESTAMP_LTZ

``` java
CREATE TABLE EventTable(
  user STRING,
  url STRING,
  ts AS PROCTIME()
) WITH ( ...
);
```



数据流转换时指定

``` java
Table table = tEnv.fromDataStream(stream, $("user"), $("url"),
$("ts").proctime());
```





### 窗口



#### 分组窗口老版本

10s 的滚动窗口

``` java
// 2.分组窗口 （老版本）
Table aggWinTable = tableEnv.sqlQuery("select user, count(1), TUMBLE_END(et, INTERVAL '10' SECOND) as entT " +
                "from clickTable group by user, TUMBLE(et, INTERVAL '10' SECOND)");
```



函数有 TUMBLE()、HOP()、SESSION()



分组窗口的功能比较有限，只支持窗口聚合，所以目前已经处于弃用(deprecated)的状态。





#### 窗口表值函数 tvt 新版本

##### 概述

窗口表值函数是 Flink 定义的多态表函数(PTF)，可以将表进行扩展后返回



tvf 提供了三个特殊的字段

“窗口起始点”(window_start)、“窗口结束点”(window_end)、“窗口时间”(window_time)











##### 滚动窗口Tumble

``` java
Table tumbleWinResTable = tableEnv.sqlQuery("select user, count(1) as cnt, "
                + "window_start as entS "
                + "from Table( "
                + "TUMBLE(TABLE clickTable, DESCRIPTOR(et), INTERVAL '10' SECOND)) "
                + "GROUP BY user, window_start, window_end");
```





``` java
// et 时间字段 10s 的窗口
TUMBLE(TABLE clickTable, DESCRIPTOR(et), INTERVAL '10' SECOND)
```



##### 滑动窗口Hop



``` java
// 1hd的滑动窗口 5min滑动一次
HOP(TABLE EventTable, DESCRIPTOR(ts), INTERVAL '5' MINUTES, INTERVAL '1' HOURS))
```



##### 累计窗口CUMULATE



每隔一段时间输出当前窗口状态 但是窗口不滑动 是滚动的



``` java
CUMULATE(TABLE EventTable, DESCRIPTOR(ts), INTERVAL '1' HOURS, INTERVAL '1' DAYS))
```

##### 会话窗口 暂未支持





## 聚合





### 分组聚合



``` java
SELECT user, COUNT(url) as cnt FROM EventTable GROUP BY user
```

函数有 SUM()、MAX()、MIN()、AVG()以及 COUNT()



防止状态无限增长耗尽资源，Flink Table API 和 SQL 可以在表 环境中配置状态的生存时间(TTL):



方式一

``` java
// 获取表环境的配置
TableConfig tableConfig = tableEnv.getConfig();
// 配置状态保持时间 
tableConfig.setIdleStateRetention(Duration.ofMinutes(60));
```

方式二

``` java
TableEnvironment tableEnv = ...
Configuration configuration = tableEnv.getConfig().getConfiguration();
configuration.setString("table.exec.state.ttl", "60 min");
```

### 窗口聚合





``` java
Table result = tableEnv.sqlQuery(
        "SELECT user, window_end AS endT, COUNT(url) AS cnt " +
        "FROM TABLE(" +
        "	TUMBLE(TABLE EventTable, " +
        "		DESCRIPTOR(ts), " +
        "		INTERVAL '1' HOUR)) " +
		"GROUP BY user, window_start, window_end"
  );
```



### 开窗聚合 over

组内排序

``` java
// over 开窗聚合
Table overResTable = tableEnv.sqlQuery("SELECT `user`, AVG(ts) OVER( " +
                                       "PARTITION BY `user` ORDER BY et ROWS BETWEEN 3 PRECEDING AND CURRENT ROW) "
                                       + "as avg_ts from clickTable");
tableEnv.toChangelogStream(overResTable).print("overResTable");
```

##### 分区，排序



#### 开窗范围

时间

当前行之前 1 小时的数据:

``` java
RANGE BETWEEN INTERVAL '1' HOUR PRECEDING AND CURRENT ROW
```

#### 行间隔

当前行+当前行的前5行

``` java
ROWS BETWEEN 5 PRECEDING AND CURRENT ROW
```





单独定义over

``` java
SELECT user, ts,
  COUNT(url) OVER w AS cnt,
  MAX(CHAR_LENGTH(url)) OVER w AS max_url
FROM EventTable
WINDOW w AS (
  PARTITION BY user
  ORDER BY ts
  ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
)
```





### TopN case

```java
package com.matt.apitest.table;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;

/**
 * @author matt
 * @create 2023-03-29 23:01
 * @desc xxx
 */
public class TopNCase {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // 1.新建表指定
        String createDDL = "CREATE TABLE clickTable (" +
                "`user` STRING, " +
                "ts BIGINT, " +
                "url STRING," +
                "et AS TO_TIMESTAMP(FROM_UNIXTIME(ts))," +
                "WATERMARK FOR et as et - INTERVAL '1' SECOND" +
                ") WITH (" +
                "'connector' = 'filesystem'," +
                "'path' = '/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt'," +
                "'format' = 'csv'" +
                ")"; // , 分割的文本文件
        tableEnv.executeSql(createDDL);

        Table topNUser = tableEnv.sqlQuery("SELECT user, cnt, row_num " +
                "FROM ( " +
                "   SELECT *, ROW_NUMBER() OVER( " +
                "       ORDER BY cnt DESC" +
                "   ) AS row_num " +
                "   FROM (SELECT user, COUNT(url) as cnt FROM clickTable GROUP BY user) " +
                ")" +
                "WHERE row_num <= 2");

        //tableEnv.toChangelogStream(topNUser).print();

        // 窗口topN 一段时间内活跃用户 top2
        // 每个窗口唯一一个结果
        Table winTopN = tableEnv.sqlQuery("SELECT user, cnt, row_num, window_end " +
                "FROM ( " +
                "   SELECT *, ROW_NUMBER() OVER( " +
                "       PARTITION BY window_start, window_end " +
                "       ORDER BY cnt DESC" +
                "   ) AS row_num " +
                "   FROM (" +
                "       SELECT user, COUNT(url) as cnt, window_start, window_end " +
                "       FROM Table (" +
                "           TUMBLE(TABLE clickTable, DESCRIPTOR(et), INTERVAL '10' SECOND) " +
                "       )" +
                "       GROUP BY user, window_start, window_end" +
                "   ) " +
                ")" +
                "WHERE row_num <= 2");
        tableEnv.toChangelogStream(winTopN).print();
        env.execute();

    }
}

```





ROW_NUMBER: 组内排序， 会添加一个排名字段

PARTITION BY: 分区，不同分区分别计算

ORDER BY 排序





## Join 连接查询



### 常规连接

和mysql 是一致的

内连接

``` sql
SELECT *
FROM Order
INNER JOIN Product
ON Order.product_id = Product.id
```



外连接

``` sql
SELECT *
FROM Order
LEFT JOIN Product
ON Order.product_id = Product.id

SELECT *
FROM Order
RIGHT JOIN Product
ON Order.product_id = Product.id

SELECT *
FROM Order
FULL OUTER JOIN Product
ON Order.product_id = Product.id
```



### 间隔连接



会增加一个时间条件限制， 如下



``` sql
ltime = rtime

ltime >= rtime AND ltime < rtime + INTERVAL '10' MINUTE

ltime BETWEEN rtime - INTERVAL '10' SECOND AND rtime + INTERVAL '5' SECOND
```



案例：下单后4h内发货

```sql
SELECT *
FROM Order o, Shipment s
WHERE o.id = s.order_id
AND o.order_time BETWEEN s.ship_time - INTERVAL '4' HOUR AND s.ship_time
```



## 函数



### 系统函数

#### 标量函数

输入一个数据 输出一个数据



##### 比较函数

```sql
value1 = value2 判断两个值相等
value1 <> value2 判断两个值不相等
value IS NOT NULL 判断 value 不为空
```

##### 逻辑函数

布尔类型拼接起来, and,or,not

```sql
boolean1 OR boolean2 布尔值 boolean1 与布尔值 boolean2 取逻辑或
boolean IS FALSE 判断布尔值 boolean 是否为 false
NOT boolean 布尔值 boolean 取逻辑非
```

##### 算术函数



numeric1 + numeric2 两数相加

POWER(numeric1, numeric2) 幂运算，取数 numeric1 的 numeric2 次方

RAND() 返回(0.0, 1.0)区间内的一个 double 类型的伪随机数



##### 字符串处理函数

string1 || string2 两个字符串的连接

UPPER(string) 将字符串 string 转为全部大写

CHAR_LENGTH(string) 计算字符串 string 的长度



##### 时间函数

- DATE string 按格式"yyyy-MM-dd"解析字符串 string，返回类型为 SQL Date
- TIMESTAMP string 按格式"yyyy-MM-dd HH:mm:ss[.SSS]"解析，返回类型为 SQL timestamp
- CURRENT_TIME 返回本地时区的当前时间，类型为 SQL time(与 LOCALTIME等价)
- INTERVAL string range 返回一个时间间隔。string 表示数值;range 可以是 DAY，MINUTE，DAT TO HOUR 等单位，也可以是 YEAR TO MONTH 这样的复合单位。如“2 年 10 个月”可以写成:INTERVAL '2-10' YEAR TO MONTH



#### 聚合函数



聚合算子有

count(*)

SUM([ ALL | DISTINCT ] expression) 对每个字段进行求和操作。默认情况下省略了关键字 ALL，表示对所有行求和;如果指定 DISTINCT，则会对数据进行去 重，每个值只叠加一次。

RANK() 返回当前值在一组值中的排名

ROW_NUMBER() 对一组值排序后，返回当前值的行号。与RANK()的 功能相似





### 自定义函数

#### 概述

支持的函数类型

- 标量函数：将输入的标量(0,1个或者多个)转换位另一个标量
- 表函数：将标量值转换成一个或多个新的行数据
- 聚合函数：将多行数据里的标量值转换成一个新的标量值;
- 表聚合函数：将多行数据里的标量值转换成一 个或多个新的行数据。





#### 基础流程



##### 1.注册函数



``` java
tableEnv.createTemporarySystemFunction("MyFunction", MyFunction.class);
```

createTemporarySystemFunction：系统函数

createTemporaryFunction:当前目录和数据库



##### 2.使用TableAPI调用函数



``` java
tableEnv.from("MyTable").select(call("MyFunction", $("myField")));
```

call: arg1(函数名) arg2(调用时传递的参数)



不注册函数直接使用



``` java
tableEnv.from("MyTable").select(call(SubstringFunction.class, $("myField")));
```



##### 3.sql中调用函数

``` java
tableEnv.sqlQuery("SELECT MyFunction(myField) FROM MyTable");
```



#### 标量函数

- 继承ScalarFunction
- 写一个public eval 方法 不是重写该方法



```java
package com.matt.apitest.table;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.flink.table.functions.ScalarFunction;

/**
 * @author matt
 * @create 2023-03-31 00:43
 * @desc xxx
 */
public class UDFTest_ScalarFunc {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // 1.新建表指定
        String createDDL = "CREATE TABLE clickTable (" +
                "`user` STRING, " +
                "ts BIGINT, " +
                "url STRING," +
                "et AS TO_TIMESTAMP(FROM_UNIXTIME(ts))," +
                "WATERMARK FOR et as et - INTERVAL '1' SECOND" +
                ") WITH (" +
                "'connector' = 'filesystem'," +
                "'path' = '/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt'," +
                "'format' = 'csv'" +
                ")"; // , 分割的文本文件
        tableEnv.executeSql(createDDL);
        tableEnv.createTemporarySystemFunction("MyHash", MyHashFunction.class);


        Table myHashTable = tableEnv.sqlQuery("SELECT user, MyHash(user) from clickTable");

        tableEnv.toDataStream(myHashTable).print();
        env.execute();
    }

    public static class MyHashFunction extends ScalarFunction {

        public int eval(String str) {
            return str.hashCode();
        }
    }

}

```





#### 表函数

表函数的输入参数也可以是 0 个、1 个或多个标量值;不同的是，它可 以返回任意多行数据。





LATERAL TABLE(MySplit(url)) as T(word, length)



LATERAL 会生成一个侧向表 然后 和主表进行连接



``` java
package com.matt.apitest.table;

import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.flink.table.functions.TableFunction;

/**
 * @author matt
 * @create 2023-03-31 01:08
 * @desc xxx
 */
public class UDFTest_TableFunction {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // 1.新建表指定
        String createDDL = "CREATE TABLE clickTable (" +
                "`user` STRING, " +
                "ts BIGINT, " +
                "url STRING," +
                "et AS TO_TIMESTAMP(FROM_UNIXTIME(ts))," +
                "WATERMARK FOR et as et - INTERVAL '1' SECOND" +
                ") WITH (" +
                "'connector' = 'filesystem'," +
                "'path' = '/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt'," +
                "'format' = 'csv'" +
                ")"; // , 分割的文本文件
        tableEnv.executeSql(createDDL);
        tableEnv.createTemporarySystemFunction("MySplit", MySplit.class);

        Table mySplitT = tableEnv.sqlQuery("SELECT user, url,word, length " +
                "from clickTable, LATERAL TABLE(MySplit(url)) as T(word, length)");

        tableEnv.toDataStream(mySplitT).print();
        env.execute();
    }

    // 自定义表函数
    public static class MySplit extends TableFunction<Tuple2<String, Integer>> {

        public void eval(String str) {
            String[] words = str.split("\\?");
            for (String w: words) {
                collect(Tuple2.of(w, w.length()));
            }
        }
    }

}

```



#### 聚合函数

会把一行或多行数据 (也就是一个表)聚合成一个标量值。这是一个标准的“多对一”的转换。





``` java
package com.matt.apitest.table;

import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.table.api.Table;
import org.apache.flink.table.api.bridge.java.StreamTableEnvironment;
import org.apache.flink.table.functions.AggregateFunction;

/**
 * @author matt
 * @create 2023-03-31 01:25
 * @desc xxx
 */
public class UTFTest_Aggfunction {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);
        StreamTableEnvironment tableEnv = StreamTableEnvironment.create(env);

        // 1.新建表指定
        String createDDL = "CREATE TABLE clickTable (" +
                "`user` STRING, " +
                "ts BIGINT, " +
                "url STRING," +
                "et AS TO_TIMESTAMP(FROM_UNIXTIME(ts))," +
                "WATERMARK FOR et as et - INTERVAL '1' SECOND" +
                ") WITH (" +
                "'connector' = 'filesystem'," +
                "'path' = '/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt'," +
                "'format' = 'csv'" +
                ")"; // , 分割的文本文件
        tableEnv.executeSql(createDDL);
        tableEnv.createTemporarySystemFunction("WeightAverage", WeightAverage.class);

        Table cc = tableEnv.sqlQuery("SELECT user, WeightAverage(ts, 1) as w_avg " +
                "from clickTable group by user");

        tableEnv.toChangelogStream(cc).print();
        env.execute();
    }
    
    public static class WeightAvgAcc {
        public long sum = 0;
        public int cnt = 0;
    }

    // 自定义聚合函数
    public static class WeightAverage extends AggregateFunction<Long, WeightAvgAcc> {

        @Override
        public Long getValue(WeightAvgAcc acc) {
            if (acc.cnt == 0) {
                return 0L;
            }
            return acc.sum / acc.cnt;
        }

        @Override
        public WeightAvgAcc createAccumulator() {
            return new WeightAvgAcc();
        }

        public void accumulate(WeightAvgAcc acc, Long iV, Integer iWeight) {
            acc.sum += iV * iWeight;
            acc.cnt += iWeight;
        }
    }

}

```



1.创建累加器

2.每来一条数据触发一次accumulate

3.获取返回数据getValue



#### 表聚合函数



用户自定义表聚合函数(UDTAGG)可以把一行或多行数据(也就是一个表)聚合成另 一张表

















``` 
./bin/sql-client.sh
```

