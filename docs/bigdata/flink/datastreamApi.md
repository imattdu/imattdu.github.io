## 概述



### 流程

- 获取执行环境get environment
- 读取数据源 source
- 数据的转换 tansfrom
- 数据的输出 sink
- 程序执行 execute





## 执行环境





### 创建执行环境

1.只能判断 如果是本地则使用本地 集群则使用集群环境

```java
StreamExecutionEnvironment env =StreamExecutionEnvironment.getExecutionEnvironment();
```

2.指定  

**createLocalEnvironment** 指定本地环境， 默认并行度cpu核数

```java
StreamExecutionEnvironment localEnv = StreamExecutionEnvironment.createLocalEnvironment();
```

**createRemoteEnvironment** 使用集群执行环境

```java
StreamExecutionEnvironment remoteEnv = StreamExecutionEnvironment.createRemoteEnvironment("host", // JobManager 主机名
1234, // JobManager 进程端口号 "path/to/jarFile.jar" // 提交给 JobManager 的 JAR 包
);
```



### 执行模式 execution mode

``` java
// 批处理环境
ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment(); 
// 流处理环境
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

1.12 批流统一, 自定义设置即可批流切换



#### 执行模式分类

流执行模式(STREAMING)，默认使用流模式



批执行模式(BATCH)， 不会持续计算的有界数据



自动模式， automatic , 程序根据输入数据的数据源是否有界， 来自动选择执行模式





#### 模式配置

Batch模式配置

```
bin/flink run -Dexecution.runtime-mode=BATCH ...
```



```
StreamExecutionEnvironment  env =StreamExecutionEnvironment.getExecutionEnvironment();
env.setRuntimeMode(RuntimeExecutionMode.BATCH);
```



### 触发执行

```java
env.execute();
```



## 源算子





![](https://raw.githubusercontent.com/imattdu/img/main/img/202303080047408.png)





使用nc 获取数据源 并行度设置为1

```
DataStream<String> stream = env.socketTextStream("localhost", 7777);
```







### 前提

有一个如下的实体类

``` java
package com.matt.apitest.model;

import java.sql.Timestamp;

public class UrlCntBO {

    public String url;
    public Long cnt;
    public Long winStart;
    public Long winEnd;

    public UrlCntBO(String url, Long cnt, Long winStart, Long winEnd) {
        this.url = url;
        this.cnt = cnt;
        this.winStart = winStart;
        this.winEnd = winEnd;
    }

    public UrlCntBO() {
    }

    @Override
    public String toString() {
        return "UrlCntBO{" +
                "url='" + url + '\'' +
                ", cnt=" + cnt +
                ", winStart=" + new Timestamp(winStart) +
                ", winEnd=" + new Timestamp(winEnd) +
                '}';
    }
}

```



- 类是公有的 public
- 有一个无参的构造方法
- 所有属性是公有的 或者提供get,set 方法
- 所有属性可序列化
- 提供一个**toString** 方法 可选



### 集合



``` java
package com.matt.apitest.source;

import com.matt.apitest.beans.Event;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.Arrays;

/**
 * @author matt
 * @create 2022-01-16 14:36
 */
public class SourceTest1_Collection {
    
    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(5);
        DataStream<Event> events = env.fromCollection(Arrays.asList(
                new Event("m1", "a1", 1672472280000L),
                new Event("m2", "a2", 1672472279514L)
        ));

        events.print("collection");
        env.execute("my");
    }
}

```



也可以使用如下方法

``` java
DataStreamSource<Event> stream2 = env.fromElements(
    new Event("Mary", "./home", 1000L),
    new Event("Bob", "./cart", 2000L)
);
```



### 文件



``` java
package com.matt.apitest.source;

import com.matt.apitest.beans.SensorReading;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

import java.util.Arrays;

/**
 * @author matt
 * @create 2022-01-16 14:54
 */
public class SourceTest2_File {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(10);
        // /Users/matt/workspace/java/stu/stu-flink/src/main/resources
        DataStream<String> dataStream = env.readTextFile("/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt");
        dataStream.print("file");

        // job name
        env.execute("f_source");
    }
}

```



如果是hdfs 可添加如下依赖

``` xml
<dependency>
   <groupId>org.apache.hadoop</groupId>
   <artifactId>hadoop-client</artifactId>
   <version>2.7.5</version>
   <scope>provided</scope>
</dependency>
```

### socket



需要将并行度设置为1

``` java
DataStream<String> stream = env.socketTextStream("localhost", 7777);
```



### kafka



添加依赖 kafka版本>=0.10.0

``` xml
<dependency>
   <groupId>org.apache.flink</groupId>
   <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
   <version>${flink.version}</version>
</dependency>
```



``` java
package com.matt.apitest.source;

import org.apache.flink.api.common.serialization.SimpleStringSchema;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.connectors.kafka.FlinkKafkaConsumer;

import java.util.Properties;

/**
 * @author matt
 * @create 2022-01-16 15:05
 */
public class SourceTest3_Kafka {

    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(1);

        Properties properties = new Properties();
        properties.setProperty("bootstrap.servers", "localhost:9092");
        properties.setProperty("group.id", "consumer-group");
        properties.setProperty("key.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        properties.setProperty("value.deserializer",
                "org.apache.kafka.common.serialization.StringDeserializer");
        
      	// 
      	properties.setProperty("auto.offset.reset", "latest");

        DataStream<String> dataStream = env.addSource( new
                FlinkKafkaConsumer<String>("first", new SimpleStringSchema(), properties));


        dataStream.print("kafka");
        env.execute("kafka_job");
    }
}

```

auto.offset.reset=latest ： 从上一个消费者提交的offset 开始消费

auto.offset.reset=earliest : 从头开始消费



### 自定义数据源

``` java
package com.matt.apitest.source;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.common.time.Time;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.streaming.api.functions.source.ParallelSourceFunction;
import org.apache.flink.streaming.api.functions.source.SourceFunction;

import java.util.Arrays;
import java.util.HashMap;
import java.util.List;
import java.util.Random;

/**
 * 自定义数据源
 *
 * @author matt
 * @create 2022-01-16 15:50
 */
public class SourceTest4_UDF {
    
    public static void main(String[] args) throws Exception {

        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        //env.setParallelism(1);

        DataStream<Event> dataStream = env.addSource(new ParallelCustomSource()).setParallelism(2);

        dataStream.print("kafka");

        // job name
        env.execute("cu");


    }

    public static class ParallelCustomSource implements ParallelSourceFunction<Event> {
        // 标志位 控制数据的产生
        private boolean running = true;

        /**
         * 功能：
         *
         * @param ctx 1
         * @author matt
         * @date 2022/1/16
         */
        @Override
        public void run(SourceContext<Event> ctx) throws InterruptedException {
            // 随机数发生器
            Random random = new Random();
            List<String> userList = Arrays.asList("matt", "jack", "lisi", "lb", "df");
            List<String> urlList = Arrays.asList("/save", "/remove", "/update", "/list", "/detail");

            while (running) {
                for (int i = 0; i < 5; i++) {
                    int randV = random.nextInt(5);
                    ctx.collect(new Event(userList.get(randV), urlList.get(randV), System.currentTimeMillis()));
                    Thread.sleep(1000L);
                }
            }
        }

        @Override
        public void cancel() {
            running = false;
        }
    }
}

```









### 支持的数据类型



#### 1.基本数据类型

Java 基本类型及其包装类，再加上 Void、String、Date、BigDecimal 和 BigInteger。

#### 2.数组类型

包括基本类型数组(PRIMITIVE_ARRAY)和对象数组(OBJECT_ARRAY)

#### 3.复合数据类型

- Java 元组类型(TUPLE):这是 Flink 内置的元组类型，是 Java API 的一部分。最多25 个字段，也就是从 Tuple0~Tuple25，不支持空字段

- Scala 样例类及 Scala 元组:不支持空字段
- 行类型(ROW):可以认为是具有任意个字段的元组,并支持空字段

- POJO:Flink 自定义的类似于 Java bean 模式的类


#### 4.辅助类型

Option、Either、List、Map 等



#### 5.泛型类型

不过如果没有按照上面 POJO 类型的要求来定义，

就会被 Flink 当作泛型类来处理。Flink 会把泛型类型当作黑盒，无法获取它们内部的属性;它 们也不是由 Flink 本身序列化的，而是由 Kryo 序列化的





pojo要求

1. 类是公共的(public)和独立的(standalone，也就是说没有非静态的内部类);
2. 类有一个公共的无参构造方法;
3. 类中的所有字段是public且非final的;或者有一个公共的getter和setter方法，这些方法需要符合 Java bean 的命名规范。





类型提示 防止泛型查除

```
.map(word -> Tuple2.of(word, 1L))
.returns(Types.TUPLE(Types.STRING, Types.LONG));


returns(new TypeHint<Tuple2<Integer, SomeType>>(){})
```







## 转换算子 transform





### 基本转换算子



``` java
package com.matt.apitest.transform;

import org.apache.flink.api.common.functions.FilterFunction;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * @author matt
 * @create 2022-01-16 17:16
 * map flatmap filter
 */
public class Base {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(16);
        DataStream<String> source = env.readTextFile("/Users/matt/workspace/java/stu/stu-flink/src/main/resources/sensor.txt");

        // 1 map string -> len(string) 1:1
        // 方-》园
        DataStream<Integer> mapStream = source.map(new MapFunction<String, Integer>() {
            @Override
            public Integer map(String s) throws Exception {
                return s.length();
            }
        });


        DataStream<Integer> lMapStream = source.map(data -> data.length());

        // flatMap 按逗号切分字端 1:n
        DataStream<String> flatMapStream = source.flatMap(new FlatMapFunction<String, String>() {
            @Override
            public void flatMap(String s, Collector<String> collector) throws Exception {
                String[] fields = s.split(",");
                for (String field : fields) {
                    collector.collect(field);
                }
            }
        });

        // 3.filter 过滤 筛选某个数据 1:[0,1]
        DataStream<String> filterStream = source.filter(new FilterFunction<String>() {
            @Override
            public boolean filter(String s) throws Exception {
                // true 要 false 不要这个数据
                return s.startsWith("sensor_1");
            }
        });

        mapStream.print("map");
        flatMapStream.print("flatMap");
        filterStream.print("filter");

        // job name
        env.execute("trans-form");
    }


}

```



### 聚合算子 aggregation



#### 按键分区





``` java
package com.matt.apitest.transform;

import com.matt.apitest.beans.Event;
import org.apache.flink.api.java.functions.KeySelector;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;

/**
 * @author matt
 * @create 2022-01-17 22:14
 */
public class Aggr {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(16);

        DataStream<Event> stream = env.fromElements(new Event("a1", "/1", 1L),
                new Event("a1", "/2", 2L),
                new Event("a2", "/1", 3L),
                new Event("a2", "/2", 2L));


        KeyedStream<Event, String> keyedSteam = stream.keyBy(new KeySelector<Event, String>() {
            @Override
            public String getKey(Event event) throws Exception {
                return event.user;
            }
        });

        keyedSteam.max("timestamp").print("1");

        // max 只更新统计的字段，其他字段都是第一个数据的值  maxBy 所有字段均更新
        stream.keyBy(e -> e.user).maxBy("timestamp")
                        .print("maxBy");
        env.execute();
    }

}

```





stream.keyBy 得到的是一个状态 不是一个算子 





#### 简单聚合



sum

min

max

minBy: min()只计算指定字段的最小值，其他字段会保留最初第一个数据的值;而 minBy()则会返回包含字段最小值的整条数据

maxBy: 和minBy类似



#### 规约聚合



更加通用化 对已有数据不断做规约



```java
package com.matt.apitest.transform;

import com.matt.apitest.beans.Event;
import com.matt.apitest.beans.SensorReading;
import org.apache.flink.api.common.functions.FlatMapFunction;
import org.apache.flink.api.common.functions.MapFunction;
import org.apache.flink.api.common.functions.ReduceFunction;
import org.apache.flink.api.java.tuple.Tuple;
import org.apache.flink.api.java.tuple.Tuple2;
import org.apache.flink.streaming.api.datastream.DataStream;
import org.apache.flink.streaming.api.datastream.KeyedStream;
import org.apache.flink.streaming.api.environment.StreamExecutionEnvironment;
import org.apache.flink.util.Collector;

/**
 * @author matt
 * @create 2022-01-17 22:48
 * 高级聚合
 */
public class Reduce {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
        env.setParallelism(16);

        DataStream<Event> stream = env.fromElements(new Event("a1", "/1", 1L),
                new Event("a1", "/2", 2L),
                new Event("a2", "/3", 3L),
                new Event("a2", "/2", 3L),
                new Event("a3", "/1", 3L),
                new Event("a2", "/3", 3L),
                new Event("a3", "/1", 3L),
                new Event("a2", "/2", 2L));

        // 2. 用户点击次数
        DataStream<Tuple2<String, Long>> userClickCntStream = stream.map(new MapFunction<Event, Tuple2<String, Long>>() {
            @Override
            public Tuple2<String, Long> map(Event event) throws Exception {
                return Tuple2.of(event.user, 1L);
            }
        }).keyBy(d -> d.f0).reduce(new ReduceFunction<Tuple2<String, Long>>() {
            @Override
            public Tuple2<String, Long> reduce(Tuple2<String, Long> a, Tuple2<String, Long> b) throws Exception {

                return Tuple2.of(a.f0, a.f1 + b.f1);
            }
        });

        // 3.当前最活跃的用户
        userClickCntStream.keyBy(d -> "xx").reduce((x, y) ->
                x.f1 >= y.f1 ? x : y
        ).print("maxActiveUser");

        // job name
        env.execute("reduce");
    }
}


```



