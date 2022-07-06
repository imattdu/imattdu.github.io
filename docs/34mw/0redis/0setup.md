







## 安装





### 注意

可参考官网安装

https://redis.io/docs/getting-started/



### mac安装



```sh
brew install redis
```

默认安装目录

/opt/homebrew/Cellar/redis/7.0.0





配置文件

/opt/homebrew/etc/redis.conf





![](https://raw.githubusercontent.com/imattdu/img/main/img/202205310300731.png)



### linux安装 todo



#### 依赖gcc环境

```sh
 yum install gcc-c++
 
 gcc --version
```

#### 基础

官网下载 -> 解压 -> 进入解压后的目录

 



#### 编译安装

```sh
make


make PREFIX=/usr/local/redis install
```

这里多了一个关键字 **`PREFIX=`** 这个关键字的作用是编译的时候用于指定程序存放的路径。比如我们现在就是指定了redis必须存放在/usr/local/redis目录。假设不添加该关键字Linux会将可执行文件存放在/usr/local/bin目录，

库文件会存放在/usr/local/lib目录。配置文件会存放在/usr/local/etc目录。其他的资源文件会存放在usr/local/share目录。这里指定号目录也方便后续的卸载，后续直接rm -rf /usr/local/redis 即可删除redis。





## 使用



### 启动服务端

```sh
//方式一：使用brew帮助我们启动软件
brew services start redis
//方式二
redis-server /opt/homebrew/etc/redis.conf
```



### 启动客户端

```sh
redis-cli -h 127.0.0.1 -p 6379

redis-cli

redis-cli -p 6379
```

### 关闭

```sh
redis-cli shutdown
```



### 测试

```sh
redis-cli
ping
```









## 配置



### 配置后台启动



1.daemonize no改成yes



```sh
daemonize yes
```

2.注释掉bind





### include

导入一个文件

```sh
include /opt/homebrew/etc/redis.conf
```



### 网络配置

#### bind

默认情况bind=127.0.0.1只能接受本机的访问请求

不写的情况下，无限制接受任何ip地址的访问

如果开启了protected-mode，那么在没有设定bind ip且没有设密码的情况下，Redis只允许接受本机的响应





#### protected-mode



保护模式，推荐设置为false





#### port

端口号





#### tcp-backlog



设置tcp的backlog，backlog其实是一个连接队列，backlog队列总和=未完成三次握手队列 + 已经完成三次握手队列。



在高并发环境下你需要一个高backlog值来避免慢客户端连接问题。



注意Linux内核会将这个值减小到/proc/sys/net/core/somaxconn的值（128），所以需要确认增大/proc/sys/net/core/somaxconn和/proc/sys/net/ipv4/tcp_max_syn_backlog（128）两个值来达到想要的效果





#### timeout

一个空闲的客户端维持多少秒会关闭，0表示关闭该功能。即永不关闭。





#### tcp-keepalive

对访问客户端的一种心跳检测，每个n秒检测一次。

单位为秒，如果设置为0，则不会进行Keepalive检测，建议设置成60 







### 通用



#### daemonize

是否为后台进程，设置为yes

守护进程，后台启动





#### pidfile

存放pid文件的位置，每个实例会产生一个不同的pid文件



```sh
pidfile /var/run/redis_6379.pid
```



#### loglevel



指定日志记录级别，Redis总共支持四个级别：debug、verbose、notice、warning，默认为**notice**

四个级别根据使用阶段来选择，生产环境选择notice 或者warning





#### logfile

日志文件名称



#### database

设定库的数量 默认16，默认数据库为0，可以使用SELECT <dbid>命令在连接上指定数据库id







### 权限

#### 密码



```sh
requirepass 123456
```

### limit





####  maxclients

设置redis同时可以与多少个客户端进行连接,默认为10000个





如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应。





#### maxmemory



最大内存







#### maxmemory-policy

淘汰规则

volatile-lru：使用LRU算法移除key，只对设置了过期时间的键；（最近最少使用）

 allkeys-lru：在所有集合key中，使用LRU算法移除key

volatile-random：在过期集合中移除随机的key，只对设置了过期时间的键

 allkeys-random：在所有集合key中，移除随机的key

volatile-ttl：移除那些TTL值最小的key，即那些最近要过期的key

noeviction：不进行移除。针对写操作，只是返回错误信息







#### maxmemory-samples





设置样本数量，LRU算法和最小TTL算法都并非是精确的算法，而是估算值，所以你可以设置样本的大小，redis默认会检查这么多个key并选择其中LRU的那个。

一般设置3到7的数字，数值越小样本越不准确，但性能消耗越小。











```sh
# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
```

