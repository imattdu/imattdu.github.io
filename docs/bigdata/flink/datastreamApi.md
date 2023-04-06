## 执行环境





### 创建执行环境

如果是本地则使用本地 集群则使用集群

```
StreamExecutionEnvironment    env =StreamExecutionEnvironment.getExecutionEnvironment();
```

使用本地执行环境， 默认并行度是cpu核数

```
StreamExecutionEnvironment localEnv = StreamExecutionEnvironment.createLocalEnvironment();
```

使用集群执行环境

```
StreamExecutionEnvironment remoteEnv = StreamExecutionEnvironment.createRemoteEnvironment("host", // JobManager 主机名
1234, // JobManager 进程端口号 "path/to/jarFile.jar" // 提交给 JobManager 的 JAR 包
);
```



### 执行模式

``` java
// 批处理环境
ExecutionEnvironment batchEnv = ExecutionEnvironment.getExecutionEnvironment(); 
// 流处理环境
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
```

1.12 批流统一

默认根据输入是否有界来决定使用

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





并行度设置为1

```
DataStream<String> stream = env.socketTextStream("localhost", 7777);
```





### 支持的数据类型



Java 基本类型及其包装类，再加上 Void、String、Date、BigDecimal 和 BigInteger。

包括基本类型数组(PRIMITIVE_ARRAY)和对象数组(OBJECT_ARRAY)



Java 元组类型(TUPLE):这是 Flink 内置的元组类型，是 Java API 的一部分。最多

25 个字段，也就是从 Tuple0~Tuple25，不支持空字段

Scala 样例类及 Scala 元组:不支持空字段

行类型(ROW):可以认为是具有任意个字段的元组,并支持空字段

POJO:Flink 自定义的类似于 Java bean 模式的类



Option、Either、List、Map 等







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

