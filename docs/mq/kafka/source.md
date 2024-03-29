

## 整体架构

### kafka工作流程



![](https://raw.githubusercontent.com/imattdu/img/main/img/202111281043347.png)



kafka中消息按照topic进行分类



topic是逻辑上的，partition是物理的



每个partition对应一个log文件，该log文件存储producer生产的数据，producer生产的数据会追加到该文件的末端，且每条数据都有自己offset，记录消费者消费数据的位置







![](https://raw.githubusercontent.com/imattdu/img/main/img/202111281049801.png)





由于生产者生产的消息会不断追加到 log 文件末尾，为防止 log 文件过大导致数据定位 效率低下，Kafka 采取了分片和索引机制，将每个 partition 分为多个 segment。每个 segment 对应两个文件——“.index”文件和“.log”文件。这些文件位于一个文件夹下，该文件夹的命名 规则为：topic 名称+分区序号。例如，first 这个 topic 有三个分区，则其对应的文件夹为 first-0,first-1,first-2。



index 和 log 文件以当前 segment 的第一条消息的 offset 命名。下图为 index 文件和 log 文件的结构示意图。



![](https://raw.githubusercontent.com/imattdu/img/main/img/202111281052358.png)



“.index”文件存储大量的索引信息，“.log”文件存储大量的数据，索引文件中的元 数据指向对应数据文件中 message 的物理偏移地址。





默认会存7天



168h







![](https://raw.githubusercontent.com/imattdu/img/main/img/202111201044790.png)



## 生产者

### 分区策略

#### 分区原因

方便在集群扩展：可以修改分区数量以适应它所在的机器

高并发：因为可以以 Partition 为单位读写了。



#### 如何分区

- 指明 partition 的情况下，直接将指明的值直接作为 partiton 值； 

- 没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值； 
- 既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后 面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 round-robin 算法。



### 数据可靠性

为保证 producer 发送的数据，能可靠的发送到指定的 topic，topic 的每个 partition 收到 producer 发送的数据后，都需要向 producer 发送 ack（acknowledgement 确认收到），如果 producer 收到 ack，就会进行下一轮的发送，否则重新发送数据。







![](https://raw.githubusercontent.com/imattdu/img/main/img/202111281058477.png)



#### 副本同步策略

| 方案                          | 优点                                                       | 缺点                                                       |
| ----------------------------- | ---------------------------------------------------------- | ---------------------------------------------------------- |
| 半数以上完成同步，就发 送 ack | 延迟低                                                     | 选举新的 leader 时，容忍 n 台 节点的故障，需要 2n+1 个副本 |
| 全部完成同步，才发送 ack      | 选举新的 leader 时，容忍 n 台 节点的故障，需要 n+1 个副 本 | 延迟高                                                     |



Kafka 选择了第二种方案，原因如下： 

1.同样为了容忍 n 台节点的故障，第一种方案需要 2n+1 个副本，而第二种方案只需要 n+1 个副本， 而 Kafka 的每个分区都有大量的数据，第一种方案会造成大量数据的冗余。 

2.虽然第二种方案的网络延迟会比较高，但网络延迟对 Kafka 的影响较小





**副本数 3**

**1 个 leader**

**2 个 follower**



#### ISR

问题：如果采用方案二同步，因为某种故障，leader不能和follower同步，那么leader就会一直不能给生产者发送ack



Leader 维护了一个动态的 in-sync replica set (ISR)，意为和 leader 保持同步的 follower 集 合。当 ISR 中的 follower 完成数据的同步之后，leader 就会给 follower 发送 ack。如果 follower 长时间 未 向 leader 同 步 数 据 ， 则 该 follower 将 被 踢 出 ISR ， 该 时 间 阈 值 由 replica.lag.time.max.ms 参数设定。Leader 发生故障之后，就会从 ISR 中选举新的 leader。



#### ack 应答机制







acks： 

0：producer 不等待 broker 的 ack，这一操作提供了一个最低的延迟，broker 一接收到还 没有写入磁盘就已经返回，当 broker 故障时有可能丢失数据；

 1：producer 等待 broker 的 ack，partition 的 leader 落盘成功后返回 ack，如果在 follower 同步成功之前 leader 故障，那么将会丢失数据；

-1（all）：producer 等待 broker 的 ack，partition 的 leader 和 follower 全部落盘成功后才 返回 ack。但是如果在 follower 同步完成后，broker 发送 ack 之前，leader 发生故障，那么会 造成数据重复。





#### 故障处理





![](https://raw.githubusercontent.com/imattdu/img/main/img/202111281116498.png)





##### 1.follower故障

follower 发生故障后会被临时踢出 ISR，待该 follower 恢复后，follower 会读取本地磁盘 记录的上次的 HW，并将 log 文件高于 HW 的部分截取掉，从 HW 开始向 leader 进行同步。 等该 follower 的 LEO 大于等于该 Partition 的 HW，即 follower 追上 leader 之后，就可以重 新加入 ISR 了。



##### 2.leader故障

leader 发生故障之后，会从 ISR 中选出一个新的 leader，之后，为保证多个副本之间的数据一致性，其余的 follower 会先将各自的 log 文件高于 HW 的部分截掉，然后从新的 leader 同步数据。





**注意：这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复。**





### Exactly Once

将服务器的 ACK 级别设置为-1，可以保证 Producer 到 Server 之间不会丢失数据，即 At Least Once 语义。相对的，将服务器 ACK 级别设置为 0，可以保证生产者每条消息只会被 发送一次，即 At Most Once 语义。



At Least Once:不丢失，可能会重复



At Most Once：不重复，可能会丢失



**At Least Once + 幂等性 = Exactly Once**



要启用幂等性，只需要将 Producer 的参数中 enable.idompotence 设置为 true 即可。Kafka 的幂等性实现其实就是将原来下游需要做的去重放在了数据上游。开启幂等性的 Producer 在 初始化的时候会被分配一个 PID，发往同一 Partition 的消息会附带 Sequence Number。而 Broker 端会对做缓存，当具有相同主键的消息提交时，Broker 只 会持久化一条。 但是 PID 重启就会变化，同时不同的 Partition 也具有不同主键，所以幂等性无法保证跨 分区跨会话的 Exactly Once。





## 消费者





### 消费方式



consumer 采用 pull（拉）模式从 broker 中读取数据。 



push（推）模式很难适应消费速率不同的消费者，因为消息发送速率是由 broker 决定的。消费者会阻塞



pull 模式不足之处是，如果 kafka 没有数据，消费者会一直轮询。针对这一点，Kafka 的消费者在消费数据时会传入一个时长参数 timeout，如果当前没有 数据可供消费，consumer 会等待一段时间之后再返回，这段时长即为 timeout。



### 分区分配策略

参考 

[分区分配](https://www.jianshu.com/p/91277c38fda6)

一个topic会有多个分区。一个消费者组会有多个消费者



#### RoundRobin

1.消费者按照字典序排序 c0 c1 c2...

2.topic按照字典序排序，并得到每个topic所有的分区，从而得到分区集合

3.遍历分区集合，同时轮询消费者

4.如果轮询的消费者没有订阅当前分区所在的topic则跳过当前消费者,g给当前分区给下一个消费者否则分配给当前消费者 同时遍历分区









------

3个Topic：T0（3个分区0, 1, 2）, T1（两个分区0, 1）, T2（4个分区0, 1, 2, 3）；
 3个consumer: C0订阅了[T0, T1]， C1订阅了[T1, T2]， C2订阅了[T2, T0]；

roundrobin结果分配结果如下：
 T0-P0分配给C0，T0-P1分配给C2，T0-P2分配给C0，
 T1-P0分配给C1，T1-P1分配给C0，
 T2-P0分配给C1，T2-P1分配给C2，T2-P2分配给C1，T0-P3分配给C2；



------



#### range

1.获取topic下的所有分区 p0 p1 p2

2.消费者按照字典序排序 c0 c1 c2

3.分区数除消费者数，得到n

4.分区数对消费者数取余，得到m

5.消费者集合中，前m个消费者有n+1分区，剩余的消费者分配n个分区





![](https://raw.githubusercontent.com/imattdu/img/main/img/202111281411561.png)











#### **注意**

**当消费者组内消费者发生变化时 分区重新分配**





### offset 维护

记录消费者消费到哪条消息



Kafka 0.9 版本之前，consumer 默认将 offset 保存在 Zookeeper 中，从 0.9 版本开始， consumer 默认将 offset 保存在 Kafka 一个内置的 topic 中，该 topic 为__consumer_offsets。

offset 消费者组 + 主题 + 分区







consumer.properties

```bash
exclude.internal.topics=false
```

#### 1.zk

```bash
./kafka-console-consumer.sh --topic __consumer_offsets --zookeeper matt05:2181 --formatter "kafka.coordinator.group.GroupMetadataManager\$OffsetsMessageFormatter" --consumer.config ../config/consumer.properties --from-beginning
```

![](https://raw.githubusercontent.com/imattdu/img/main/img/20211121172409.png)





![](https://raw.githubusercontent.com/imattdu/img/main/img/20211121173630.png)





![](https://raw.githubusercontent.com/imattdu/img/main/img/20211121172223.png)



#### 2、mq



![](https://raw.githubusercontent.com/imattdu/img/main/img/20211121175135.png)









```bash
[console-consumer-39623,bigdata,0]::[OffsetMetadata[4,NO_METADATA],CommitTime 1637488435475,ExpirationTime 1637574835475]
[console-consumer-39623,bigdata,1]::[OffsetMetadata[4,NO_METADATA],CommitTime 1637488435475,ExpirationTime 1637574835475]
```









### 验证一个分区只可以被一个消费者消费







matt05 matt06



```bash
vi consumer.properties
```



matt05 matt06

```bash
group.id=matt
```



5 6 启动消费者



```bash
./kafka-console-consumer.sh --zookeeper matt05:2181 --topic first --consumer.config ../config/consumer.properties
```



7生产者



```bash
./kafka-console-producer.sh --broker-list matt05:9092 --topic first
```



## kafka



1）顺序写磁盘 Kafka 的 producer 生产数据，要写入到 log 文件中，写的过程是一直追加到文件末端， 为顺序写。官网有数据表明，同样的磁盘，顺序写能到 600M/s，而随机写只有 100K/s。这 与磁盘的机械机构有关，顺序写之所以快，是因为其省去了大量磁头寻址的时间。



2）零拷贝

![](https://raw.githubusercontent.com/imattdu/img/main/img/20210720224609.png)







![](https://raw.githubusercontent.com/imattdu/img/main/img/20210720224642.png)





## zk



Kafka 集群中有一个 broker 会被选举为 Controller，负责管理集群 broker 的上下线，所 有 topic 的分区副本分配和 leader 选举等工作。 Controller 的管理工作都是依赖于 Zookeeper 的





![](https://raw.githubusercontent.com/imattdu/img/main/img/202111281436680.png)









zk:监听节点

## 事务



Kafka 从 0.11 版本开始引入了事务支持。事务可以保证 Kafka 在 Exactly Once 语义的基 础上，生产和消费可以跨分区和会话，要么全部成功，要么全部失败。



### producer

Transaction ID，并将 Producer 获得的PID 和Transaction ID 绑定。并写入内部topic。



TransactionL：用户给定的

### consumer

消费者 事务较弱

   

用户可以修改offset
