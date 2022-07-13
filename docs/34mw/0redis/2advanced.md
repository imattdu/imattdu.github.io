

## 发布订阅





### 概述

发布者：发送消息

订阅者:接收信息





### redis发布订阅





订阅管道c1

```sh
subscribe c1
```

在管道c1发布消息

```sh
publish c1 1
```



## 新数据类型





### bitmaps

存储01组成的数组





#### 命令

##### setbit



![](https://raw.githubusercontent.com/imattdu/img/main/img/202206282336522.png)



offset从0开始

```sh
setbit b1 0 0
```





##### getbit

getbit key offset

```sh
getbit b1 0
```



##### bitcount



bitcount key start end



start,end 均包含 start->end之间1的个数

-1：最后一位

-2：倒数第二位



```sh
bitcount b1 0 3
```



##### bitop



bitop and(or/not/xor)  destkey k1 k2 k3 ....





and：&&

or：或者

not：非

xor:异或



```sh
bitop and b3 b1 b2
```



### HyperLogLog



#### 概述

统计某个key 不重复元素的个数



#### 优势

并不会存储输入的元素，而是根据输入元素计算基数





HyperLogLog只需要12KB 就可以计算 2e64不同元素的基数







{1, 2, 3, 3, 4, 5} 

基数集

{1, 2, 3, 4, 5}

基数

5





#### 命令

##### pfadd

```sh
pfadd key v1 v2 ....


pfadd p1 1 2 3

```



##### pfcount

计算多个key的基数

```sh
pfcount k1 k2 k3 ....


pfcount p1


pfcount p1 p2
```





##### pfmerge

将多个key进行合并

```sh
pfmerge destkey k1 k2 k3




pfmerge p3 p2 p1
```



### Geospatial 



#### 概述

经纬度



#### 命令



##### geoadd 添加

有效的经度从 -180 度到 180 度。有效的纬度从 -85.05112878 度到 85.05112878 度。

```sh

geoadd key 经度 纬度 成员 [经度 纬度 成员]


geoadd g1 1 1 a1
```



##### geopos 定位



获取指定成员的经度 纬度

```sh

geopos key member [member ...]


127.0.0.1:6379> geopos g1 a1
1) 1) "0.99999994039535522"
   2) "0.99999945914297683"
```



##### geodist 获取俩个位置的直线距离

m 表示单位为米[默认值]。

km 表示单位为千米。

mi 表示单位为英里。

ft 表示单位为英尺。

```sh
geodist key member1 member2 [m | km | ft | mi]


geodist g1 a1 a2 m
```

##### georadius 给定经纬度为中心 找到一定半径内的元素





```sh
georadius key 经度 纬度 radius m | km | ft | mi



georadius g1 1 1 1000 km
```









## redis 事务





### redis事务定义



Redis事务是一个单独的隔离操作：事务中所有命令都会序列化、按顺序的执行。事务执行过程中，不会被可他客户端发送的命令打断。









### multi exec discard



从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行。

组队的过程中可以通过discard来放弃组队。



```sh
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set 1 1
QUEUED
127.0.0.1:6379(TX)> set 2 2
QUEUED
127.0.0.1:6379(TX)> exec
1) OK
2) OK
127.0.0.1:6379>
```



### 如果发生错误

组队中某个命令出现了报告错误，执行时整个的所有队列都会被取消。

如果执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会回滚。





### 锁

#### 悲观锁

**悲观锁(Pessimistic Lock)**, 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block直到它拿到锁



#### 乐观锁 



**乐观锁(Optimistic Lock),** 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**。Redis就是利用这种check-and-set机制实现事务的。







### watch unwatch





在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在事务**执行之前这个(****或这些) key** **被其他命令所改动，那么事务将被打断。**





```sh
127.0.0.1:6379> watch 1
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379(TX)> set 1 1
QUEUED
127.0.0.1:6379(TX)> exec
(nil)
127.0.0.1:6379>
```



unwatch 取消对key的监视







### redis事务特性



Ø 单独的隔离操作 

事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 

Ø 没有隔离级别的概念 

队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行

Ø 不保证原子性 

事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 















## 持久化

### rdb





#### what

在指定的时间间隔内将内存中的数据集快照写入磁盘， 也就是行话讲的Snapshot快照，它恢复时是将快照文件直接读到内存里





#### source



##### fork todo

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到 一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。



 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能 如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。**RDB**的缺点是最后一次持久化后的数据可能丢失。







#### config



##### 指定文件位置

redis.conf

```sh
dir /opt/homebrew/var/db/redis/
```

##### 配置触发策略



![](https://raw.githubusercontent.com/imattdu/img/main/img/202207091120910.png)



不设置save指令，或者给save传入空字符串





###### 手动触发



save ：save时只管保存，其它不管，全部阻塞。手动保存。不建议。

bgsave：Redis会在后台异步进行快照操作，快照同时还可以响应客户端请求。





##### stop-writes-on-bgsave-error



当Redis无法写入磁盘的话，直接关掉Redis的写操作。推荐yes.





##### rdbcompression 

使用压缩算法 , LZF



##### rdbchecksum 检查完整性







#### 优点缺点



##### 优点

l 节省磁盘空间

l 恢复速度快



##### 缺点



虽然Redis在fork时使用了**写时拷贝技术**,但是如果数据庞大时还是比较消耗性能。

在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

















### aof



#### what



以**日志**的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(**读操作不记录**)， **只许追加文件但不可以改写文件**，redis启动之初会读取该文件重新构建数据，





换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作





#### source

##### 如何同步

- 客户端的请求写命令会被append追加到AOF缓冲区内；
- AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作sync同步到磁盘的AOF文件中；
- AOF文件大小超过重写策略或手动重写时，会对AOF文件rewrite重写，压缩AOF文件容量
- Redis服务重启时，会重新load加载AOF文件中的写操作达到数据恢复的目的





![](https://raw.githubusercontent.com/imattdu/img/main/img/202207091601939.png)







##### 压缩

###### what

AOF采用文件追加方式，文件会越来越大为避免出现此种情况，新增了重写机制, 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩， 只保留可以恢复数据的最小指令集.可以使用命令bgrewriteaof



AOF文件持续增长而过大时，会fork出一条新进程来将文件重写(也是先写临时文件最后再rename)，redis4.0版本后的重写，是指上就是把rdb 的快照，以二级制的形式附在新的aof头部，作为已有的历史数据，替换掉原来的流水账操作。





###### source



- bgrewriteaof触发重写，判断是否当前有bgsave或bgrewriteaof在运行，如果有，则等待该命令结束后再继续执行。
- 主进程fork出子进程执行重写操作，保证主进程不会阻塞。
- 子进程遍历redis内存中数据到临时文件，客户端的写请求同时写入aof_buf缓冲区和aof_rewrite_buf重写缓冲区保证原AOF文件完整以及新AOF文件生成期间的新的数据修改动作不会丢失。
- 1).子进程写完新的AOF文件后，向主进程发信号，父进程更新统计信息。2).主进程把aof_rewrite_buf中的数据写入到新的AOF文件。
- 使用新的AOF文件覆盖旧的AOF文件，完成AOF重写。





###### config

**写配置**

如果 no-appendfsync-on-rewrite=yes ,不写入aof文件只写入缓存，用户请求不会阻塞，但是在这段时间如果宕机会丢失这段时间的缓存数据。（降低数据安全性，提高性能）

​    如果 no-appendfsync-on-rewrite=no, 还是会把数据往磁盘里刷，但是遇到重写操作，可能会发生阻塞。（数据安全，但是性能降低）

触发机制，何时重写







**触发策略配置**

Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发

重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。 

auto-aof-rewrite-percentage：设置重写的基准值，文件达到100%时开始重写（文件是原来重写后文件的2倍时触发）

auto-aof-rewrite-min-size：设置重写的基准值，最小文件64MB。达到这个值开始重写。

例如：文件达到70MB开始重写，降到50MB，下次什么时候开始重写？100MB

系统载入时或者上次重写完毕时，Redis会记录此时AOF大小，设为base_size,



如果Redis的AOF当前大小>= base_size +base_size*100% (默认)且当前大小>=64mb(默认)的情况下，Redis会对AOF进行重写。 









#### config



###### 开启配置

可以在redis.conf中配置文件名称，默认为 appendonly.aof

AOF文件的保存路径，同RDB的路径一致。



默认aof不开启，如果aof,rdb同时开启，则使用aof





###### 同步频率配置

appendfsync always

始终同步，每次Redis的写入都会立刻记入日志；性能较差但数据完整性比较好

appendfsync everysec

每秒同步，每秒记入日志一次，如果宕机，本秒的数据可能丢失。

appendfsync no

redis不主动进行同步，把同步时机交给操作系统。





#### use

###### 恢复

正常恢复

- 修改默认的appendonly no，改为yes
- 将有数据的aof文件复制一份保存到对应目录(查看目录：config get dir)
- 恢复：重启redis然后重新加载



异常恢复

- 修改默认的appendonly no，改为yes
- 如遇到AOF文件损坏，通过/usr/local/bin/redis-check-aof--fix appendonly.aof进行修复
- 备份被写坏的AOF文件
- 恢复：重启redis，然后重新加载





#### summary



##### advantages

- 备份机制更稳健，丢失数据概率更低。



##### disadvantages



- 比起RDB占用更多的磁盘空间。
- 恢复备份速度要慢。
- 每次读写都同步的话，有一定的性能压力。









## 主从复制





### what

读写分离，master写，复制到slaver, slaver进行读





### use

#### 基础

##### 配置



从单机中的redis.conf复制一份redis.conf



新建redis6379.conf,redis6380.conf,redis6381.conf

填入如下内容，修改**pidfile,port,dbfilename**

```sh
include /opt/homebrew/etc/myredis/redis.conf
pidfile "/var/run/redis_6379.pid"
port 6379
dbfilename dump6379.rdb
```

##### 启动

启动三台redis服务器

```sh
❯ redis-server ./redis6379.conf
❯ redis-server ./redis6380.conf
❯ redis-server ./redis6381.conf
```



##### 查看

```sh
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:0
master_failover_state:no-failover
master_replid:08f03d154f528b0980ccb908bff473e966a19509
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```



##### 从机配置 主机不配置

6380,6381配置

```sh
slaveof 127.0.0.1 6379
```



##### 注意

从机写数据就会报错

```sh
127.0.0.1:6381> set 1 1
(error) READONLY You can't write against a read only replica.
```

主机挂掉重启即可

从机重启需要重新配置slaveof 127.0.0.1 6379，也可以写在配置文件中





#### 常见的三种主从模式



##### 一主二从

![](https://raw.githubusercontent.com/imattdu/img/main/img/202207122357675.png)





##### 薪火相传



![](https://raw.githubusercontent.com/imattdu/img/main/img/202207122359620.png)





降低了master复制时的写压力

主机挂掉 从机仍然是从机



##### 反客为主



基于薪火相传

当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。



使用如下命令

```sh
slaveof no one 
```







#### 哨兵模式



##### what

**反客为主的自动版**，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库



##### use



使用一主二仆， master:6379, slaver:6380,6381



配置文件下新建sentinel.conf

```sh
sentinel monitor mymaster 127.0.0.1 6379 1
```



启动哨兵

```sh
redis-sentinel  ./sentinel.conf 
```







选举规则

1

master挂掉，则根据下列条件从slaver中选举一个master

优先级在redis.conf中默认：slave-priority 100，值越小优先级越高

偏移量是指获得原主机数据最全的

每个redis实例启动后都会随机生成一个40位的runid



2sentine向原主服务的从服务发送slaveof命令，发送复制数据



3.如果一个master已经下线又重新上线，sentinel会向其发送slaveof命令，让其成为新主的从







### source

#### 复制原理





- Slave启动成功连接到master后会发送一个sync命令
- Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步
- 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步
- 但是只要是重新连接master,一次完全同步（全量复制)将被自动执行













## 集群





查看集群信息

 ```sh
 cluster nodes
 ```

![](https://raw.githubusercontent.com/imattdu/img/main/img/202206191121531.png)







```sh
127.0.0.1:6379> set s1 1
-> Redirected to slot [15224] located at 192.168.199.222:6381
OK
```









```sh
192.168.199.222:6381> mset name{user} matt age{user} 12
-> Redirected to slot [5474] located at 192.168.199.222:6380
```





只能看自己槽位 别的看不到





```sh
127.0.0.1:6380> cluster nodes
2fcb1f4118c3ba1d0f00cc0e859e812fae552942 192.168.199.222:6379@16379 master,fail - 1655622346443 1655622343385 1 disconnected
73d2f2aa22351cb220cb4d70da9ddcbcf0066e40 192.168.199.222:6381@16381 master - 0 1655622363871 3 connected 10923-16383
2a7c5c0eb6a9d43298245584f1c1fad4420b3aaf 192.168.199.222:6390@16390 slave 73d2f2aa22351cb220cb4d70da9ddcbcf0066e40 0 1655622364000 3 connected
23b19e64cf536fdffc11b7748e01d9b232f30045 192.168.199.222:6391@16391 master - 0 1655622364903 7 connected 0-5460
282398ec24fbcf9743a50d8389dcc536c2dee6a0 192.168.199.222:6389@16389 slave 6511a15d81a488b8e776f73e655d27bd36004e5c 0 1655622365933 2 connected
6511a15d81a488b8e776f73e655d27bd36004e5c 192.168.199.222:6380@16380 myself,master - 0 1655622362000 2 connected 5461-10922
```





![](https://raw.githubusercontent.com/imattdu/img/main/img/202206191509291.png)









分布式锁



a

1 获取锁

3 释放锁

2 卡注



2执行 释放锁 会把别的线程的锁释放















```sh
127.0.0.1:6379> acl list
1) "user default on nopass ~* &* +@all"
```







```sh
 acl setuser matt
```

