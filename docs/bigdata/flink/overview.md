





## 基本概念

### 是什么

Apache Flink 是一个框架和分布式处理引擎，用于对无界和有界数据流进行状态计算



### 特点

- 高吞吐和低延迟。每秒处理数百万个事件，毫秒级延迟
- 结果的准确性。Flink 提供了事件时间(event-time)和处理时间(processing-time) 语义。对于乱序事件流，事件时间语义仍然能提供一致且准确的结果。
- 精确一次(exactly-once)的状态一致性保证。
- 可以连接到最常用的存储系统，如 Apache Kafka、Apache Cassandra、Elasticsearch、JDBC、Kinesis 和(分布式)文件系统，如 HDFS 和 S3。
- 高可用。本身高可用的设置，加上与K8s，YARN和Mesos的紧密集成，再加上从故障中快速恢复和动态扩展任务的能力，Flink 能做到以极少的停机时间 7×24 全天候运行。
- 能够更新应用程序代码并将作业(jobs)迁移到不同的 Flink 集群，而不会丢失应用程序的状态。



### **Flink** VS **Spark Streaming**



数据模型

-  Spark 底层数据模型是弹性分布式数据集(RDD)，Spark Streaming 进行微批处理的底层 接口 DStream，实际上处理的也是一组组小批数据 RDD 的集合

-  Flink 的基本数据模型是数据流(DataFlow)，以及事件(Event)序列

运行时架构

-   Spark 是批计算，将 任务对应的DAG 划分为不同的 阶段stage，一个完成后经过 shuffle 再进行下一阶段的计算
-   Flink 是标准的流式执行模式，一个事件在一个节点处理完后可以直接发往下一个节点进行处理





```
# 保持链接
nc -lk 777
```







无界数据

数据是源源不断产生



有界数据

有明确定义数据的开始和结束

