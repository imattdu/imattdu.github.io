## producer



### java

#### 基础

Kafka 的 Producer 发送消息采用的是异步发送的方式。

在消息发送的过程中，涉及到了 两个线程——main 线程和 Sender 线程，以及一个线程共享变量——RecordAccumulator。

 main 线程将消息发送给 RecordAccumulator，Sender 线程不断从 RecordAccumulator 中拉取 消息发送到 Kafka broker。





batch.size：只有数据积累到 batch.size 之后，sender 才会发送数据。 

linger.ms：如果数据迟迟未达到 batch.size，sender 等待 linger.time 之后就会发送数据。



![](https://raw.githubusercontent.com/imattdu/img/main/img/202209251716577.png)









#### 带回调函数

exception == null 消息发送成功





#### 同步

一条消息发送之后，会阻塞当前线程，直至返回 ack。

future.Get()









## consumer





### java



#### 基础

KafkaConsumer：需要创建一个消费者对象，用来消费数据 

ConsumerConfig：获取所需的一系列配置参数 

ConsuemrRecord：每条数据都要封装成一个 ConsumerRecord 对象 



为了使我们能够专注于自己的业务逻辑，Kafka 提供了自动提交 offset 的功能。  

enable.auto.commit：是否开启自动提交 offset 功能 

auto.commit.interval.ms：自动提交 offset 的时间间隔





```xml
<dependency>
  <groupId>org.apache.kafka</groupId>
  <artifactId>kafka-clients</artifactId>
  <version>0.11.0.0</version>
</dependency>
```





#### 手动提交

 commitSync（同步提交）和 commitAsync（异步提交）



两者的相同点是，都会将本次 poll 的一批数据最高的偏移量提交；不同点是， commitSync 阻塞当前线程，一直到提交成功，并且会自动失败重试（由不可控因素导致， 也会出现提交失败）；而 commitAsync 则没有失败重试机制，故有可能提交失败。



#### 问题

无论是同步提交还是异步提交 offset，都有可能会造成数据的漏消费或者重复消费。先提交 offset 后消费，有可能造成数据的漏消费；而先消费后提交 offset，有可能会造成数据 的重复消费。