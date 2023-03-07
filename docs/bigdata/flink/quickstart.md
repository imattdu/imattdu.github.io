

## helloword



### wc

1.新建maven项目



2.编写pom.xml



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.matt</groupId>
    <artifactId>stu-flink</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <flink.version>1.13.0</flink.version>
        <java.version>1.8</java.version>
        <scala.binary.version>2.12</scala.binary.version>
        <slf4j.version>1.7.30</slf4j.version>
    </properties>

    <dependencies>
        <!-- flink 使用到scala 组件-->
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-java</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-streaming-java_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>


        <!-- 日志 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>${slf4j.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-to-slf4j</artifactId>
            <version>2.14.0</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <!--打包工具-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```



3.编写日志配置文件

log4j.properties

```properties
log4j.rootLogger=error, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n
```



运行时参数指定

![](https://raw.githubusercontent.com/imattdu/img/main/img/202203100130741.png)



4.具体代码



![](https://raw.githubusercontent.com/imattdu/img/main/img/202303052323889.png)







## 部署



### mac-Standalone 模式





#### 修改文件所有者

```sh
# 查看当前用户
whoami
# 查看用户组
id matt

# 用户名：组名
chown -R matt:staff /opt/software

chown -R matt:staff /opt/module


```

查看更多用户相关信息

https://blog.csdn.net/qq_26129413/article/details/109675386





#### 下载解压

```sh
tar -zxvf flink-1.13.0-bin-scala_2.12.tgz -C /opt/module
```

#### 配置

##### 修改 flink/conf/flink-conf.yaml 文件 jobmanager

如果是单机安装默认即可，集群安装配置某台主机

```sh
jobmanager.rpc.address: localhost
```



##### 修改 /conf/workers 文件 taskmanager

从机机器列表

```sh
localhost
```

如果是集群安装可以配置为

```sh
matt06
matt07
```

如果是集群安装需要把*flink*同步到其他机器





##### 其他配置

- jobmanager.memory.process.size:对JobManager进程可使用到的全部内存进行配置，包括 JVM 元空间和其他开销，默认为 1600M，可以根据集群规模进行适当调整。
- taskmanager.memory.process.size:对TaskManager进程可使用到的全部内存进行配置，包括 JVM 元空间和其他开销，默认为 1600M，可以根据集群规模进行适当调整。
- taskmanager.numberOfTaskSlots:对每个TaskManager能够分配的Slot数量进行配置，默认为 1，可根据 TaskManager 所在的机器能够供给 Flink 的 CPU 数量决定。所谓Slot 就是 TaskManager 中具体运行一个任务所分配的计算资源。
-  parallelism.default:Flink任务执行的默认并行度，优先级低于代码中进行的并行度配置和任务提交交时使用参数指定的并行度数量。



#### 启动

在jobmanger机器上进行启动

bin

```sh
start-cluster.sh


stop-cluster.sh
```



#### 提交任务

依赖的jar 包也打进去

``` xml
<build>
        <plugins>
            <!--打包工具-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>3.0.0</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
```





##### ui 提交

http://localhost:8081/#/overview



![](https://raw.githubusercontent.com/imattdu/img/main/img/202212241450389.png)







![](https://raw.githubusercontent.com/imattdu/img/main/img/202303062325420.png)





```sh
❯ ./bin/flink list
Waiting for response...
------------------ Running/Restarting Jobs -------------------
24.12.2022 15:11:40 : a21e498f6c313c5be5941d45aaef4ed1 : Flink Streaming Job (RUNNING)
--------------------------------------------------------------
No scheduled jobs.
❯ ./bin/flink cancel a21e498f6c313c5be5941d45aaef4ed1
Cancelling job a21e498f6c313c5be5941d45aaef4ed1.
Cancelled job a21e498f6c313c5be5941d45aaef4ed1.
```



##### 命令行提交

可以通过命令行提交也可以ui进行提交

-m ip

```sh
./bin/flink run -c com.matt.wc.StreamWordCount -p 1 stu-flink-1.0-SNAPSHOT.jar --host localhost --port 777
```



### 部署模式

#### 会话模式

启动一个flink集群 ，提交作业， 所有作业公用一个集群



#### 单作业模式

由客户端运行 应用程序，然后启动集群，作业被交给 JobManager，进而分发给 TaskManager 执行。作业完成后，集群就会关闭，所有资源也会释放。



Flink 本身无法直接这样运行，所以单作业模式一般需要借助一些资源管理框架来启动集群，比如 YARN、Kubernetes。



#### 应用模式

不需要客户端，直接把应用提交到 JobManger 上运行我们需要为每一个提交的应用单独启动一个 JobManager，也就是创建一个集群。这个 JobManager 只为执行这一个应用而存在，执行结束之后 JobManager 也就关闭了。





### 独立模式部署

#### 会话模式

先启动集群会提交作业



#### 单作业模式

Flink 本身无法直接以单作业方式启动集群，一般需要借助一些资 源管理平台。所以 Flink 的独立(Standalone)集群并不支持单作业模式部署。





#### 应用模式部署



应用模式下不会提前创建集群，所以不能调用 start-cluster.sh 脚本。我们可以使用同样在 bin 目录下的 standalone-job.sh 来创建一个 JobManager。



具体步骤如下:

(1)进入到 Flink 的安装路径下，将应用程序的 jar 包放到 lib/目录下。

```
cp ./FlinkTutorial-1.0-SNAPSHOT.jar lib/
```

(2)执行以下命令，启动 JobManager。

```
./bin/standalone-job.sh start --job-classname com.atguigu.wc.StreamWordCount
```

这里我们直接指定作业入口类，脚本会到 lib 目录扫所有的 jar 包。



 (3)同样是使用 bin 目录下的脚本，启动 TaskManager。

``` 
./bin/taskmanager.sh start
```

(4)如果希望停掉集群，同样可以使用脚本，命令如下。

``` 
./bin/standalone-job.sh stop
./bin/taskmanager.sh stop
```



### **TODO yarn/k8s部署**









## 运行架构





![](https://raw.githubusercontent.com/imattdu/img/main/img/202212251601292.png)







### 构成





#### JobManager

JobManager 是一个 Flink 集群中任务管理和调度的核心，是控制应用执行的主进程。每个应用都应该被唯一的 JobManager 所控制执行。当然，在高可用(HA)的场景下可能会出现多个 JobManager;这时只有一个是正在运行的领导节点(leader)，其他都是备用节点(standby)。





##### JobMaster

在作业提交时，JobMaster 会先接收到要执行的应用。这里所说“应用”一般是客户端提交 交来的，包括:Jar 包，数据流图(dataflow graph)，和作业图(JobGraph)。

JobMaster 会把 JobGraph 转换成一个物理层面的数据流图，这个图被叫作“执行图” (ExecutionGraph)，它包含了所有可以并发执行的任务。JobMaster 会向资源管理器 (ResourceManager)发出请求，申请执行任务必要的资源。一旦它获取到了足够的资源，就会将执行图分发到真正运行它们的 TaskManager 上。



而在运行过程中，JobMaster 会负责所有需要中央协调的操作，比如说检查点(checkpoints)的协调。



##### 资源管理器(ResourceManager)

主要负责资源的分配和管理。在 Flink 集群中只有一个。所谓“资源”， 主要是指 TaskManager 的任务槽(task slots)。任务槽就是 Flink 集群中的资源调配单元，包含 了机器用来执行计算的一组 CPU 和内存资源。每一个任务(Task)都需要分配到一个 slot 上 执行。



Flink 的 ResourceManager，针对不同的环境和资源管理平台(比如 Standalone 部署，或者 YARN)，有不同的具体实现。在 Standalone 部署时，因为 TaskManager 是单独启动的(没有 Per-Job 模式)，所以 ResourceManager 只能分发可用 TaskManager 的任务槽，不能单独启动新 TaskManager。



而在有资源管理平台时，就不受此限制。当新的作业申请资源时，ResourceManager 会将 有空闲槽位的 TaskManager 分配给 JobMaster。如果 ResourceManager 没有足够的任务槽，它 还可以向资源提供平台发起会话，请求提供启动 TaskManager 进程的容器。另外， ResourceManager 还负责停掉空闲的 TaskManager，释放计算资源。





##### Dispatcher 

主要负责提供一个 REST 接口，用来提交应用，并且负责为每一个新提交的作 业启动一个新的 JobMaster 组件。Dispatcher 也会启动一个 Web UI，用来方便地展示和监控作业执行的信息。Dispatcher 在架构中并不是必需的，在不同的部署模式下可能会被忽略掉。





#### TaskManager



TaskManager 是 Flink 中的工作进程，数据流的具体计算就是它来做的，所以也被称为 “Worker”。Flink 集群中必须至少有一个 TaskManager;



当然由于分布式计算的考虑，通常会 有多个 TaskManager 运行，每一个 TaskManager 都包含了一定数量的任务槽(task slots)。Slot

是资源调度的最小单位，slot 的数量限制了 TaskManager 能够并行处理的任务数量。 启动之后，TaskManager 会向资源管理器注册它的 slots;收到资源管理器的指令后， TaskManager就会将一个或者多个槽位提供给JobMaster调用，JobMaster就可以分配任务来执行了。



在执行过程中，TaskManager 可以缓冲数据，还可以跟其他运行同一应用的 TaskManager交换数据。





### 任务提交流程



![](https://raw.githubusercontent.com/imattdu/img/main/img/202212251612766.png)





#### 抽象

1. 一般情况下，由客户端(App)通过分发器提供的 REST 接口，将作业提交给JobManager。
2. 由分发器启动 JobMaster，并将作业(包含 JobGraph)提交给 JobMaster。
3.  JobMaster 将 JobGraph 解析为可执行的 ExecutionGraph，得到所需的资源数量，然后向资源管理器请求资源(slots)。
4.  资源管理器判断当前是否由足够的可用资源;如果没有，启动新的 TaskManager。
5.  TaskManager 启动之后，向 ResourceManager 注册自己的可用任务槽(slots)。 
6. 资源管理器通知 TaskManager 为新的作业提供 slots。
7. TaskManager 连接到对应的 JobMaster，提供 slots。 
8. JobMaster 将需要执行的任务分发给 TaskManager。 
9. TaskManager 执行任务，互相之间可以交换数据



部署模式不同，有些步骤可能不相同





#### 独立模式

会话模式和应用模式 俩者是相似的



**不会启动 TaskManager，而且直接向已有的 TaskManager 要求资源**



#### yarn 集群

##### 会话模式



![](https://raw.githubusercontent.com/imattdu/img/main/img/202303070923233.png)





![](https://raw.githubusercontent.com/imattdu/img/main/img/202303070923633.png)

在会话模式下，我们需要先启动一个 YARN session，这个会话会创建一个 Flink 集群。





这里只启动了 JobManager，而 TaskManager 可以根据需要动态地启动。在 JobManager 内部，由于还没有提交作业，所以只有 ResourceManager 和 Dispatcher 在运行





1. 客户端通过 REST 接口，将作业提交给分发器。
2. 分发器启动 JobMaster，并将作业(包含 JobGraph)提交给 JobMaster。 
3. JobMaster 向资源管理器请求资源(slots)。
4. 资源管理器向 YARN 的资源管理器请求 container 资源。
5. YARN 启动新的 TaskManager 容器。
6. TaskManager 启动之后，向 Flink 的资源管理器注册自己的可用任务槽。
7. 资源管理器通知 TaskManager 为新的作业提供 slots。 
8. TaskManager 连接到对应的 JobMaster，提供 slots。
9. JobMaster 将需要执行的任务分发给 TaskManager，执行任务。



##### 单作业模式



1. 客户端将作业提交给 YARN 的资源管理器，这一步中会同时将 Flink 的 Jar 包和配置 上传到 HDFS，以便后续启动 Flink 相关组件的容器。
2. YARN的资源管理器分配Container资源，启动Flink JobManager，并将作业提交给 JobMaster。这里省略了 Dispatcher 组件。
3. JobMaster 向资源管理器请求资源(slots)。
4. 资源管理器向 YARN 的资源管理器请求 container 资源。
5. YARN 启动新的 TaskManager 容器。
6. TaskManager 启动之后，向 Flink 的资源管理器注册自己的可用任务槽。
7. 资源管理器通知 TaskManager 为新的作业提供 slots。 
8. TaskManager 连接到对应的 JobMaster，提供 slots。
9. JobMaster 将需要执行的任务分发给 TaskManager，执行任务。



可见，区别只在于 JobManager 的启动方式，以及省去了分发器。当第 2 步作业提交交给 JobMaster，之后的流程就与会话模式完全一样了。





##### 应用模式

应用模式与单作业模式的提交流程非常相似，只是初始提交给 YARN 资源管理器的不再是具体的作业，而是整个应用。一个应用中可能包含了多个作业，这些作业都将在 Flink 集群中启动各自对应的 JobMaster。





### 一些重要概念











### 任务调度原理

















### 并行度



![](https://raw.githubusercontent.com/imattdu/img/main/img/202212282224443.png)









一个特定算子的 子任务（subtask）的个数被称之为其并行度（parallelism）。

 一般情况下，一个 stream 的并行度，可以认为就是其所有算子中最大的并行度。







### TaskManager 和 Slots



​		Flink 中每一个 worker(TaskManager)都是一个 JVM 进程，它可能会在独立的线 程上执行一个或多个 subtask。为了控制一个 worker 能接收多少个 task，worker 通 过 task slot 来进行控制（一个 worker 至少有一个 task slot）。 



​		每个 task slot 表示 TaskManager 拥有资源的一个固定大小的子集。假如一个 TaskManager 有三个 slot，那么它会将其管理的内存分成三份给各个 slot。资源 slot 化意味着一个 subtask 将不需要跟来自其他 job 的 subtask 竞争被管理的内存，取而 代之的是它将拥有一定数量的内存储备。需要注意的是，这里不会涉及到 CPU 的隔 离，slot 目前仅仅用来隔离 task 的受管理的内存。 

​		

​		通过调整 task slot 的数量，允许用户定义 subtask 之间如何互相隔离。如果一个 TaskManager 一个 slot，那将意味着每个 task group 运行在独立的 JVM 中（该 JVM 可能是通过一个特定的容器启动的），而一个 TaskManager 多个 slot 意味着更多的 subtask 可以共享同一个 JVM。而在同一个 JVM 进程中的 task 将共享 TCP 连接（基 于多路复用）和心跳消息。它们也可能共享数据集和数据结构，因此这减少了每个 task 的负载。

![](https://raw.githubusercontent.com/imattdu/img/main/img/202203092335800.png)



Flink 中每一个 TaskManager 都是一个JVM进程，它可能会在独立的线程上执 行一个或多个子任务 

 为了控制一个 TaskManager 能接收多少个 task， TaskManager 通过 task slot 来进行控制（一个 TaskManager 至少有一个 slot）







![](https://raw.githubusercontent.com/imattdu/img/main/img/202203092336258.png)





​		默认情况下，Flink 允许子任务共享 slot，即使它们是不同任务的子任务（前提 是它们来自同一个 job）。 这样的结果是，一个 slot 可以保存作业的整个管道。 



​		Task Slot 是静态的概念，是指 TaskManager 具有的并发执行能力，可以通过 参数 taskmanager.numberOfTaskSlots 进行配置；而并行度 parallelism 是动态概念， 即 TaskManager 运行程序时实际使用的并发能力，可以通过参数 parallelism.default 进行配置。 



​		也就是说，假设一共有 3 个 TaskManager，每一个 TaskManager 中的分配 3 个 TaskSlot，也就是每个 TaskManager 可以接收 3 个 task，一共 9 个 TaskSlot，如果我 们设置 parallelism.default=1，即运行程序默认的并行度为 1，9 个 TaskSlot 只用了 1 个，有 8 个空闲，因此，设置合适的并行度才能提高效率。







### 并行子任务的分配







![子任务](https://raw.githubusercontent.com/imattdu/img/main/img/202203092339776.png)









### 程序与数据流（DataFlow）



所有的Flink程序都是由三部分组成的： Source 、Transformation 和 Sink。Source 负责读取数据源，Transformation 利用各种算子进行处理加工，Sink负责输出



在运行时，Flink上运行的程序会被映射成“逻辑数据流”（dataflows），它包 含了这三部分 每一个dataflow以一个或多个sources开始以一个或多个sinks结束。dataflow 类似于任意的有向无环图（DAG）  在大部分情况下，程序中的转换运算（transformations）跟dataflow中的算子 （operator）是一一对应的关系





![](https://raw.githubusercontent.com/imattdu/img/main/img/202203092348844.png)









### 执行图（ExecutionGraph）



Flink 中的执行图可以分成四层：StreamGraph -> JobGraph -> ExecutionGraph -> 物理执行图 

➢ StreamGraph：是根据用户通过 Stream API 编写的代码生成的最初的图。用来 表示程序的拓扑结构。

 ➢ JobGraph：StreamGraph经过优化后生成了 JobGraph，提交给 JobManager 的数据结构。主要的优化为，将多个符合条件的节点 chain 在一起作为一个节点 

➢ ExecutionGraph：JobManager 根据 JobGraph 生成ExecutionGraph。 ExecutionGraph是JobGraph的并行化版本，是调度层最核心的数据结构。

 ➢ 物理执行图：JobManager 根据 ExecutionGraph 对 Job 进行调度后，在各个 TaskManager 上部署 Task 后形成的“图”，并不是一个具体的数据结构。









![](https://raw.githubusercontent.com/imattdu/img/main/img/202203092352306.png)









### 数据传输形式





一个程序中，不同的算子可能具有不同的并行度 

算子之间传输数据的形式可以是 one-to-one (forwarding) 的模式也可以是 redistributing 的模式，具体是哪一种形式，取决于算子的种类 

➢ One-to-one：stream维护着分区以及元素的顺序（比如source和map之间）。 这意味着map 算子的子任务看到的元素的个数以及顺序跟 source 算子的子任务 生产的元素的个数、顺序相同。map、fliter、flatMap等算子都是one-to-one 的对应关系。 

➢ Redistributing：stream的分区会发生改变。每一个算子的子任务依据所选择的 transformation发送数据到不同的目标任务。例如，keyBy 基于 hashCode 重 分区、而 broadcast 和 rebalance 会随机重新分区，这些算子都会引起 redistribute过程，而 redistribute 过程就类似于 Spark 中的 shuffle 过程。





### 任务链（Operator Chains）



Flink 采用了一种称为任务链的优化技术，可以在特定条件下减少本地通信的开销。为了满足任务链的要求，必须将两个或多个算子设为相同 的并行度，并通过本地转发（local forward）的方式进行连接 

 相同并行度的 one-to-one 操作，Flink 这样相连的算子链接在一起形 成一个 task，原来的算子成为里面的 subtask 

并行度相同、并且是 one-to-one 操作，两个条件缺一不可





好处

它能减少线 程之间的切换和基于缓存区的数据交换，在减少时延的同时提升吞吐量。链接的行为可以在编程 API 中进行指定。





```java
 //基于数据流进行转换计算
        DataStream<Tuple2<String, Integer>> resultStream = inputDataStream.flatMap(new WordCount.MyFlatMapper())
                .keyBy(0)
                .sum(1).setParallelism(2).startNewChain();
        // 和前后都不合并任务
        // .disableChaining();


        // 开始一个新的任务链合并 前面断开后面不断开
        // .startNewChain()
```





![](https://raw.githubusercontent.com/imattdu/img/main/img/202203092355735.png)













```java
// 禁用算子链 当前算子不和前后算子合并
.map(word -> Tuple2.of(word, 1L)).disableChaining();
// 从当前算子开始新链
.map(word -> Tuple2.of(word, 1L)).startNewChain()
```



