



## 终端操作kafka



如果启动失败 查看logs/server.log



### 查看所有topic



```sh
./kafka-topics.sh --zookeeper localhost:2181 --list
```



### 创建topic

```sh
./kafka-topics.sh --zookeeper localhost:2181 --create --replication-factor  1 --partitions 2 --topic first
```

选项
说明：
--topic 定义 topic名
--replication-factor 定义副本数
--partitions 定义分区数



**副本数量 不能超过机器数**

分区可以大于机器数量

 

### 删除topic

```sh
./kafka-topics.sh --zookeeper localhost:2181 --delete --topic first
```

**需要server.properties中设置 delete.topic.enable=true否则只是标记删除。**







### 查看topic详情



```sh
./kafka-topics.sh --zookeeper 127.0.0.1:2181 --describe --topic first
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202111180115815.png)









### 发送消息



```sh
./kafka-console-producer.sh --broker-list localhost:9092 --topic first
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202111180125374.png)





### 消费消息

可以连接zookeeper

```sh
./kafka-console-consumer.sh --zookeeper localhost:2181 --topic first
```

会有警告 

![](https://raw.githubusercontent.com/imattdu/img/main/img/202111180130466.png)





```sh
./kafka-console-consumer.sh --bootstrap-server matt05:9092 --topic first
```

从最开始的位置消费

```sh
./kafka-console-consumer.sh --bootstrap-server matt05:9092 --from-beginning --topic first
```





### 修改分区数



```shell
./kafka-topics.sh --zookeeper localhost:2181 --alter --topic first --partitions 3
```











