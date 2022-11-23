



​            

## 地址

[http://kafka.apache.org/downloads.html](http://kafka.apache.org/downloads.html)







## mac





### 地址

https://kafka.apache.org/downloads.html

0.11.0.0

![](https://raw.githubusercontent.com/imattdu/img/main/img/202209241431775.png)





### 安装

#### 解压缩

```sh
tar -zxvf kafka_2.11-0.11.0.0.tgz -C /opt/module
```

#### 修改文件名称

```sh
mv kafka_2.11-0.11.0.0 kafka
```



### 配置

#### 创建日志目录

```sh
cd kafka


mkdir logs
```



#### 修改配置文件

```sh
cd config

vim server.properties
```





```sh
#broker 的全局唯一编号，不能重复
broker.id=0
#删除 topic 功能使能
delete.topic.enable=true


#kafka 运行日志存放的路径
log.dirs=/opt/module/kafka/logs

#配置连接 Zookeeper 集群地址
zookeeper.connect=matt05:2181,matt06:2181
```





#### 配置环境变量



vim .zshrc.pre-oh-my-zsh

```sh
vim .zshrc.pre-oh-my-zsh
```



```sh
#KAFKA_HOME
export KAFKA_HOME=/opt/module/kafka
export PATH=$PATH:$KAFKA_HOME/bin
```



### zk 安装

#### 地址



https://archive.apache.org/dist/zookeeper/zookeeper-3.5.7/





![](https://raw.githubusercontent.com/imattdu/img/main/img/202209241449105.png)

#### 解压缩 + 修改文件名



```sh
tar -zxvf apache-zookeeper-3.5.7-bin.tar.gz -C /opt/module
```



```sh
mv apache-zookeeper-3.5.7-bin/ zookeeper-3.5.7
```



#### 创建日志目录

```sh
mkdir zkData
```



#### 配置文件修改

```sh
cp zoo_sample.cfg zoo.cfg
```



```sh
dataDir=/opt/module/zookeeper-3.5.7/zkData
```



#### 启动





```sh
zkServer.sh start

zkServer.sh stop

zkServer.sh status
```







启动kafka

```sh
./bin/kafka-server-start.sh -daemon ./config/server.properties
```







### 启动



#### 启动,安装目录bin目录下

```bash
./kafka-server-start.sh -daemon ../config/server.properties
```



#### 关闭

```bash
./kafka-server-stop.sh stop
```





## linux



### 规划

matt05 zk kafka

matt06 zk kafka

matt07 zk kafka





### 解压

```bash
tar -zxvf kafka_2.11-0.11.0.0.tgz -C /opt/module/
```

### 重命名



```bash
mv kafka_2.11-0.11.0.0/ kafka
```

**2.scanla版本**

**0.11kafka**版本

### 在安装目录下创建data文件夹



默认日志放在logs 

data:存储数据

```bash
mkdir data
```

### 修改配置文件

```bash
cd config/
vi server.properties
```



```sh
# 改 broker 的全局唯一编号，不能重复 
broker.id=0
# 改 删除 topic 功能使能 
delete.topic.enable=true 

# 改 kafka 运行日志存放的路径 日志目录
log.dirs=/opt/module/kafka/data 

# 改 配置连接 Zookeeper 集群地址 
zookeeper.connect=zookeeper.connect=192.168.96.135:2181,192.168.96.136:2181,192.168.96.137:2181


```



### 分发安装包

```bash
xsync ./kafka/
```

**记得修改机器的broker.id 因为他是惟一的**





### 启动,安装目录bin目录下， 分别进入三台机器

```bash
./kafka-server-start.sh -daemon ../config/server.properties
```



### 关闭,安装目录bin目录下

```bash
./kafka-server-stop.sh stop
```



使用时可能会无法连接kafka，在server.properties进行如下配置

**<Connection to node 0 could not be established. Broker may not be available.>**

```bash
advertised.listeners=PLAINTEXT://192.168.96.128:9092
```



























