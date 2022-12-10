## overview



### what

**MapReduce 是一个分布式运算程序的编程框架**，是用户开发“基于 Hadoop 的数据分析应用”的核心框架。

MapReduce 核心功能是将用户编写的业务逻辑代码和自带默认组件整合成一个完整的 分布式运算程序，并发运行在一个 Hadoop 集群上。



### ad disad

#### ad

1.MapReduce 易于编程

​	它简单的实现一些接口，就可以完成一个分布式程序，

2.良好的扩展性

​	可以动态增加机器

3.高容错性

​	其中一台机器挂了，它可以把上面的计算任务转移到另外一个节点上运行， 不至于这个任务运行失败



#### disad

1.不擅长实时计算

​	MapReduce 无法像 MySQL 一样，在毫秒或者秒级内返回结果。

2.不擅长流式计算

​	流式计算的输入数据是动态的，而 MapReduce 的输入数据集是静态的，不能动态变化。 这是因为 MapReduce 自身的设计特点决定了数据源必须是静态的。



3.不擅长 DAG（有向无环图）计算

​	多个应用程序存在依赖关系，后一个应用程序的输入为前一个的输出。在这种情况下， MapReduce 并不是不能做，而是使用后，每个 MapReduce 作业的输出结果都会写入到磁盘， 会造成大量的磁盘 IO，导致性能非常的低下。



### 核心思想





![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226095146.png)







1.分布式的运算程序往往需要分成至少 2 个阶段。 

2.第一个阶段的 MapTask 并发实例，完全并行运行，互不相干。 

3.第二个阶段的 ReduceTask 并发实例互不相干，但是他们的数据依赖于上一个阶段 的所有 MapTask 并发实例的输出。 

4.MapReduce 编程模型只能包含一个 Map 阶段和一个 Reduce 阶段，如果用户的业 务逻辑非常复杂，那就只能多个 MapReduce 程序，串行运行。









### MapReduce进程



（1）MrAppMaster：负责整个程序的过程调度及状态协调。

（2）MapTask：负责 Map 阶段的整个数据处理流程。 

（3）ReduceTask：负责 Reduce 阶段的整个数据处理流程。





### 序列化类型

![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226095554.png)





### mapreduce编码规范



1．Mapper阶段 

（1）用户自定义的Mapper要继承自己的父类 

（2）Mapper的输入数据是KV对的形式（KV的类型可自定义） 

（3）Mapper中的业务逻辑写在map()方法中 

（4）Mapper的输出数据是KV对的形式（KV的类型可自定义） 

（5）map()方法（MapTask进程）对每一个调用一次



2．Reducer阶段 

（1）用户自定义的Reducer要继承自己的父类

（2）Reducer的输入数据类型对应Mapper的输出数据类型，也是KV 

（3）Reducer的业务逻辑写在reduce()方法中 

（4）ReduceTask进程对每一组相同k的组调用一次reduce()方法 



3．Driver阶段 

相当于YARN集群的客户端，用于提交我们整个程序到YARN集群，提交的是 封装了MapReduce程序相关运行参数的job对象





## 实操

### demo



#### 前提



需要首先配置好 HADOOP_HOME 变量以及 Windows 运行依赖（参考hdfs）





```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.matt.stu</groupId>
    <artifactId>stu-mapreduce</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
    </properties>


    <dependencies>
        <dependency>
            <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>3.1.3</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.30</version>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <!--依赖的jar包也打进去-->
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
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





log4j.properties

```properties
log4j.rootLogger=INFO, stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p [%c] - %m%n
log4j.appender.logfile=org.apache.log4j.FileAppender
log4j.appender.logfile.File=target/spring.log
log4j.appender.logfile.layout=org.apache.log4j.PatternLayout
log4j.appender.logfile.layout.ConversionPattern=%d %p [%c] - %m%n
```

#### 具体实现

统计文本中每个单词出现的次数

map:单词的切割

reduce:单词的统计

driver:任务的配置 和提交



<img src="https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226100623.png" style="zoom:50%;" />







### hadoop集群运行

#### 打包依赖

```xml
<build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.1</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
            <!--依赖的jar包也打进去-->
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
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

#### 运行



**不带依赖的jar包**

```sh
hadoop jar wc.jar com.matt.mapreduce.wordcount2.WordCountDriver /input /output
```





#### 默认案例

```go
/opt/module/hadoop-3.1.3/share/hadoop/mapreduce
```



```go
sz hadoop-mapreduce-examples-3.1.3.jar
```

 



### 更多案例

参考

https://github.com/imattdu/stu-mapreduce







## mapreduce 原理





### src-提交job



![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226114616.png)



1.建立连接



![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226114715.png)





![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226114831.png)





835建立连接



2.提交job

839提交任务 **->**



![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226114918.png)



(1)81创建给集群提交数据的 Stag 路径



(2)90获取 jobid ，并创建 Job 路径

![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226115020.png)



(3)121拷贝 jar 包到集群

(4)124计算切片，生成切片规划文件

![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226115044.png)



(5)153向 Stag 路径写 XML 配置文件



(6)155提交 Job,返回提交状态

![](https://raw.githubusercontent.com/imattdu/img/main/img/2021/12/26/20211226115108.png)





![](https://raw.githubusercontent.com/imattdu/img/main/img/20210809232800.png)







### src-FileInputFormat



#### src

1.程序先找到你数据存储的目录。 

2.开始遍历处理（规划切片）目录下的每一个文件 

3.遍历第一个文件ss.txt 

​	(1)获取文件大小fs.sizeOf(ss.txt) 

​	(2)计算切片大小 computeSplitSize(Math.max(minSize,Math.min(maxSize,blocksize)))=blocksize=128M 

​	(3)默认情况下，切片大小=blocksize

​	(4)开始切，形成第1个切片：ss.txt—0:128M 第2个切片ss.txt—128:256M 第3个切片ss.txt—256M:300M （每次切片时，都要判断切完剩下的部分是否大于块的1.1倍，不大于1.1倍就划分一块切片） 

​	(5)将切片信息写到一个切片规划文件中

​	(6)整个切片的核心过程在getSplit()方法中完成 

​	(7)inputSplit只记录了切片的元数据信息，比如起始位置、长度以及所在的节点列表等。 

4.提交切片规划文件到YARN上，YARN上的MrAppMaster就可以根据切片规划文件计算开启MapTask个数。



#### 切片机制

1.简单地按照文件的内容长度进行切片 

2.切片大小，默认等于Block大小 

3.切片时不考虑数据集整体，而是逐个针对每一个文件单独切片



#### 切片口径

```java
Math.max(minSize, Math.min(maxSize, blockSize)); 

mapreduce.input.fileinputformat.split.minsize=1 默认值为1 
mapreduce.input.fileinputformat.split.maxsize= Long.MAXValue 默认值Long.MAXValue
```



#### 获取切片信息

```java
// 获取切片的文件名称
String name = inputSplit.getPath().getName();
// 根据文件类型获取切片信息
FileSplit inputSplit = (FileSplit) context.getInputSplit();
```



### FileInputFormat 常见实现类



#### TextInputFormat

TextInputFormat 是默认的 FileInputFormat 实现类。按行读取每条记录。键是存储该行在整个文件中的起始字节偏移量， LongWritable 类型。值是这行的内容，不包括任何行终止 符（换行符和回车符），Text 类型



#### CombineTextInputFormat

CombineTextInputFormat 用于小文件过多的场景，它可以将多个小文件从逻辑上规划到 一个切片中，这样，多个小文件就可以交给一个 MapTask 处理。





##### 切片



虚拟存储

将输入目录下所有文件大小，依次和设置的 setMaxInputSplitSize 值比较，如果不大于设置的最大值，逻辑上划分一个块。如果输入文件大于设置的最大值且大于两倍， 那么以最大值切割一块；当剩余数据大小超过设置的最大值且不大于最大值 2 倍，此时 将文件均分成 2 个虚拟存储块（防止出现太小切片）。



切片

1.判断虚拟存储的文件大小是否大于 setMaxInputSplitSize 值，大于等于则单独 形成一个切片。 

2.如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。







![](https://raw.githubusercontent.com/imattdu/img/main/img/20210810002436.png)









### mapreduce 工作流程





![](https://raw.githubusercontent.com/imattdu/img/main/img/202211261558521.png)













![](https://raw.githubusercontent.com/imattdu/img/main/img/202211261550423.png)





1.MapTask 收集我们的 map()方法输出的 kv 对，放到内存缓冲区中 

2.从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件 

3.多个溢出文件会被合并成大的溢出文件 

4.在溢出过程及合并的过程中，都要调用 Partitioner 进行分区和针对 key 进行排序 

5.ReduceTask 根据自己的分区号，去各个 MapTask 机器上取相应的结果分区数据 

6.ReduceTask 会抓取到同一个分区的来自不同 MapTask 的结果文件，ReduceTask 会将这些文件再进行合并（归并排序） 

7.合并成大文件后，Shuffle 的过程也就结束了，后面进入 ReduceTask 的逻辑运算过 程（从文件中取出一个一个的键值对 Group，调用用户自定义的 reduce()方法）





### shuffle



map之后reduce之前 -> shuffle





![](https://raw.githubusercontent.com/imattdu/img/main/img/202211261557264.png)





## 排序





对于MapTask，它会将处理的结果暂时放到环形缓冲区中，当环形缓冲区使 用率达到一定阈值后，再对缓冲区中的数据进行一次快速排序，并将这些有序数 据溢写到磁盘上，而当数据处理完毕后，它会对磁盘上所有文件进行归并排序。



 对于ReduceTask，它从每个MapTask上远程拷贝相应的数据文件，如果文件大 小超过一定阈值，则溢写磁盘上，否则存储在内存中。如果磁盘上文件数目达到 一定阈值，则进行一次归并排序以生成一个更大文件；如果内存中文件大小或者 数目超过一定阈值，则进行一次合并后将数据溢写到磁盘上。当所有数据拷贝完 毕后，ReduceTask统一对内存和磁盘上的所有数据进行一次归并排序。







（1）部分排序 MapReduce根据输入记录的键对数据集排序。保证输出的每个文件内部有序。 

（2）全排序 最终输出结果只有一个文件，且文件内部有序。实现方式是只设置一个ReduceTask。但该方法在 处理大型文件时效率极低，因为一台机器处理所有文件，完全丧失了MapReduce所提供的并行架构。 

（3）辅助排序：（GroupingComparator分组） 在Reduce端对key进行分组。应用于：在接收的key为bean对象时，想让一个或几个字段相同（全部 字段比较不相同）的key进入到同一个reduce方法时，可以采用分组排序。 （4）二次排序 在自定义排序过程中，如果compareTo中的判断条件为两个即为二次排序。









待记录

https://blog.csdn.net/zyw0101/article/details/126626837









## src-mapreduce





### maptask

![](https://raw.githubusercontent.com/imattdu/img/main/img/202211261636433.png)





1.Read 阶段：MapTask 通过 InputFormat 获得的 RecordReader，从输入 InputSplit 中 解析出一个个 key/value。 2.Map 阶段：该节点主要是将解析出的 key/value 交给用户编写 map()函数处理，并 产生一系列新的 key/value。 3.Collect 收集阶段：在用户编写 map()函数中，当数据处理完成后，一般会调用 OutputCollector.collect()输出结果。在该函数内部，它会将生成的 key/value 分区（调用 Partitioner），并写入一个环形内存缓冲区中。 

4.Spill 阶段：即“溢写”，当环形缓冲区满后，MapReduce 会将数据写到本地磁盘上， 生成一个临时文件。需要注意的是，将数据写入本地磁盘之前，先要对数据进行一次本地排序，并在必要时对数据进行合并、压缩等操作。



溢写阶段详情： 

步骤 1：利用快速排序算法对缓存区内的数据进行排序，排序方式是，先按照分区编号 Partition 进行排序，然后按照 key 进行排序。这样，经过排序后，数据以分区为单位聚集在 一起，且同一分区内所有数据按照 key 有序。 

步骤 2：按照分区编号由小到大依次将每个分区中的数据写入任务工作目录下的临时文 件 output/spillN.out（N 表示当前溢写次数）中。如果用户设置了 Combiner，则写入文件之 前，对每个分区中的数据进行一次聚集操作。 

步骤 3：将分区数据的元信息写到内存索引数据结构 SpillRecord 中，其中每个分区的元 信息包括在临时文件中的偏移量、压缩前数据大小和压缩后数据大小。如果当前内存索引大 小超过 1MB，则将内存索引写到文件 output/spillN.out.index 中。 



5.Merge 阶段：当所有数据处理完成后，MapTask 对所有临时文件进行一次合并， 以确保最终只会生成一个数据文件。 当所有数据处理完后，MapTask 会将所有临时文件合并成一个大文件，并保存到文件 output/file.out 中，同时生成相应的索引文件 output/file.out.index。 在进行文件合并过程中，MapTask 以分区为单位进行合并。对于某个分区，它将采用多 轮递归合并的方式。每轮合并 mapreduce.task.io.sort.factor（默认 10）个文件，并将产生的文 件重新加入待合并列表中，对文件排序后，重复以上过程，直到最终得到一个大文件。 让每个 MapTask 最终只生成一个数据文件，可避免同时打开大量文件和同时读取大量 小文件产生的随机读取带来的开销。









### reduce 工作机制

![](https://raw.githubusercontent.com/imattdu/img/main/img/202211261634165.png)





1.Copy 阶段：ReduceTask 从各个 MapTask 上远程拷贝一片数据，并针对某一片数 据，如果其大小超过一定阈值，则写到磁盘上，否则直接放到内存中。 

2.Sort 阶段：在远程拷贝数据的同时，ReduceTask 启动了两个后台线程对内存和磁 盘上的文件进行合并，以防止内存使用过多或磁盘上文件过多。按照 MapReduce 语义，用 户编写 reduce()函数输入数据是按 key 进行聚集的一组数据。为了将 key 相同的数据聚在一 起，Hadoop 采用了基于排序的策略。由于各个 MapTask 已经实现对自己的处理结果进行了 局部排序，因此，ReduceTask 只需对所有数据进行一次归并排序即可。 

3.Reduce 阶段：reduce()函数将计算结果写到 HDFS 上。







![](https://raw.githubusercontent.com/imattdu/img/main/img/202211261638540.png)









```java
=================== ReduceTask ===================
if (isMapOrReduce()) //reduceTask324 行，提前打断点
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202211261638605.png)









## 压缩





### why



#### ad disad

压缩的优点：以减少磁盘 IO、减少磁盘存储空间。 

压缩的缺点：增加 CPU 开销。



运算密集型的 Job，少用压缩 

IO 密集型的 Job，多用压缩





### 压缩算法介绍

![](https://raw.githubusercontent.com/imattdu/img/main/img/202211261648535.png)



![](https://raw.githubusercontent.com/imattdu/img/main/img/202211261648600.png)





算法 	原始大小	压缩大小	压缩速度 	解压速度 

gzip 	8.3GB 	1.8GB	 17.5MB/s 	58MB/s 

bzip2 	8.3GB 	1.1GB 	2.4MB/s 	9.5MB/s

 LZO 	8.3GB 	2.9GB 	49.3MB/s 	74.6MB/s







### 位置选择

1.输入端采用压缩

无须显示指定使用的编解码方 式。Hadoop自动检查文件扩 展名，如果扩展名能够匹配， 就会用恰当的编解码方式对文 件进行压缩和解压。 企业开发中如何选择：为了减 少MapTask和ReduceTask之间的网络 IO。重点考虑压缩和解压缩快的 LZO、Snappy。 看需求： 如果数据永久保存，考虑压缩 率比较高的Bzip2和Gzip。 如果作为下一个MapReduce输 入，需要考虑数据量和是否支持切 企业开发： 1）数据量小于块大小，重点 考虑压缩和解压缩速度比较快 的LZO/Snappy 2）数据量非常大，重点考虑 支持切片的Bzip2和LZO





2.map输出

企业开发中如何选择：为了减少MapTask和ReduceTask之间的网络 IO。重点考虑压缩和解压缩快的 LZO、Snappy。



3.reduce 输出

看需求： 如果数据永久保存，考虑压缩 率比较高的Bzip2和Gzip。 如果作为下一个MapReduce输 入，需要考虑数据量和是否支持切片。







### 压缩配置

### 算法

| 压缩格式 | 压缩格式 对应的编码/解码器                 |
| -------- | ------------------------------------------ |
| DEFLATE  | org.apache.hadoop.io.compress.DefaultCodec |
| gzip     | org.apache.hadoop.io.compress.GzipCodec    |
| bzip2    | org.apache.hadoop.io.compress.BZip2Codec   |
| LZO      | com.hadoop.compression.lzo.LzopCodec       |
| Snappy   | org.apache.hadoop.io.compress.SnappyCodec  |

#### hadoop启用



| 参数                                                         | 默认值                                           | 阶段         | 建议                                               |
| ------------------------------------------------------------ | ------------------------------------------------ | ------------ | -------------------------------------------------- |
| io.compression.codecs （在 core-site.xml 中配置）            | 无，这个需要在命令行输入 hadoop checknative 查看 | 输入压缩     | Hadoop 使用文件扩展 名判断是否支持某种 编解码器    |
| mapreduce.map.output.compress（在 mapred-site.xml 中 配置）  | false                                            | mapper 输出  | 这个参数设为 true 启 用压缩                        |
| mapreduce.map.output.compress.codec（在 mapredsite.xml 中配置） | org.apache.hadoop.io.com press.DefaultCodec      | mapper 输出  | 企业多使用 LZO 或 Snappy 编解码器在此 阶段压缩数据 |
| mapreduce.output.fileoutpu tformat.compress（在 mapred-site.xml 中配置） | false                                            | reducer 输出 | 这个参数设为 true 启 用压缩                        |
| mapreduce.output.fileoutpu tformat.compress.codec（在 mapred-site.xml 中配置） | org.apache.hadoop.io.com press.DefaultCodec      | reducer 输出 | 使用标准工具或者编 解码器，如 gzip 和 bzip2        |
|                                                              |                                                  |              |                                                    |







源码支持

BZip2Codec、DefaultCodec



```java
// 开启 map 端输出压缩
conf.setBoolean("mapreduce.map.output.compress", true);
// 设置 map 端输出压缩方式
conf.setClass("mapreduce.map.output.compress.codec",
BZip2Codec.class,CompressionCodec.class);
```







```java
// 设置 reduce 端输出压缩开启
FileOutputFormat.setCompressOutput(job, true);
// 设置压缩的方式
 FileOutputFormat.setOutputCompressorClass(job, BZip2Codec.class);
```

