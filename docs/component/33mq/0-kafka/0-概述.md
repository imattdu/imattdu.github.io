

## 概述

### 是什么

Kafka 是一个分布式的基于发布/订阅模式的消息队列（Message Queue），主要应用于大数据实时处理领域



### mq模式

1.发布/订阅（1:1 1:n） 拉取 缺点可能需要长轮训

2.队列主动推 造成消费者阻塞







kafka：拉取 长轮训 设置超时时间



### 好处

解耦：允许用户独立扩展生产和消费的流程（消费有故障也不会影响生产）

削峰：解决生产和消费速度不一致，防止消费速度过慢导致阻塞





### 架构



![](https://raw.githubusercontent.com/imattdu/img/main/img/202111170123975.png)

   







- Producer ：消息生产者，就是向kafka broker 发消息的客户端；
- Consumer ：消息消费者，向kafka broker 取消息的客户端；
- Consumer Group （CG）：消费者组，消费者组可以有一个或多个消费者，一个topic的一个分区只能由消费者组内一个消费者消费
- Broker ：一台kafka 服务器就是一个broker。一个集群由多个broker 组成。
- Topic ：可以理解为一个队列，生产者和消费者面向的都是一个topic；
- Partition：一个topic可由多个分区组成
- Replica：副本，leader follower
- leader 每个分区多个副本的“主”，生产者发送数据的对象，以及消费者消费数据的对 象都是 leader。
- follower：每个分区多个副本中的“从”，实时从 leader 中同步数据，保持和 leader 数据 的同步。

