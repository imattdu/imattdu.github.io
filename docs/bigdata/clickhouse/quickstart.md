## overview



### what

ClickHouse 是俄罗斯的 Yandex 于 2016 年开源的列式存储数据库(DBMS)，使用 C++ 语言编写，主要用于在线分析处理查询(OLAP)，能够使用 SQL 查询实时生成分析数据报告。



### features

#### 1.列式存储

行式存储

```
1 张三 18 2 张二 18 3 张一 20
```

列式存储

```
1 2 3 张三 李四 王五 18 22 34
```

case:

如果想查所有年龄，行式需要遍历所有数据（有很多字段是不关心的），列式存储只需要提取处特定的列即可



列式存储好处：

- 对于列的聚合，计数，求和等统计操作原因优于行式存储。
- 由于某一列的数据类型都是相同的，针对于数据存储更容易进行数据压缩，每一列 选择更优的数据压缩算法，大大提高高了数据的压缩比重。



#### 2.DBMS 的功能

几乎覆盖了标准 SQL 的大部分语法，包括 DDL 和 DML，以及配套的各种函数，用户管理及权限管理，数据的备份与恢复



#### 3.多样化引擎

ClickHouse 和 MySQL 类似，把表级的存储引擎插件化，根据表的不同需求可以设定不同的存储引擎。目前包括合并树、日志、接口和其他四大类 20 多种引擎。



#### 4.高吞吐写入能力

ClickHouse 采用类 LSM Tree 的结构，数据写入后定期在后台 Compaction（压缩）。通过类 LSM tree 的结构，ClickHouse 在数据导入时全部是顺序 append 写，写入后数据段不可更改，在后台 compaction 时也是多个段 merge sort 后顺序写回磁盘。顺序写的特性，充分利用了磁盘的吞吐能力，即便在 HDD 上也有着优异的写入性能。



官方公开 benchmark 测试显示能够达到 50MB-200MB/s 的写入吞吐能力，按照每行 100Byte 估算，大约相当于 50W-200W 条/s 的写入速度。



#### 5.数据分区和线程级并行

partition index granularity

数据分区，每个分区根据索引粒度划分为多个，然后通过多个 CPU 核心分别处理其中的一部分来实现并行数据处理。



但是有一个弊端就是对于单条查询使用多 cpu，就不利于同时并发多条查询









```sh
e4c9c994ffc3   centos:centos7   "/bin/bash"   2 weeks ago   Exited (255) 37 seconds ago   0.0.0.0:8123->8123/tcp, 0.0.0.0:9000->9000/tcp   elegant_williams
```





## quick start



### install

#### docker - centos

##### 1.取消文件数限制

hard: 上限

```sh
sudo vim /etc/security/limits.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
```



```sh
sudo vim /etc/security/limits.d/20-nproc.conf
* soft nofile 65536
* hard nofile 65536
* soft nproc 131072
* hard nproc 131072
```



##### 2.禁用 SELINUX

```sh
sudo vim /etc/selinux/config
SELINUX=disabled
```

重启

##### 3 install

``` sh
docker run -p 9000:9000 -p 8123:8123 -it centos:centos7 /bin/bash


yum install sudo
yum install yum
yum install vim

sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://packages.clickhouse.com/rpm/clickhouse.repo
sudo yum install -y clickhouse-server clickhouse-client
```

##### 4 conf

```sh
sudo vim /etc/clickhouse-server/config.xml


<listen_host>0.0.0.0</listen_host>
```

<listen_host>0.0.0.0</listen_host> 注释打开

允许非本机可访问



##### 5.use

```
sudo /etc/init.d/clickhouse-server start
clickhouse-client -m  # or "clickhouse-client --password" if you set up a password.
```



#### docker - origin ubuntu

```
docker run -d -p 8123:8123 -p 9000:9000 --name ck23.2 --ulimit nofile=262144:262144 clickhouse/clickhouse-server:23.2
```

该镜像是ubuntu

```
apt-get update
apt-get install vim -y
apt-get install sudo

不需要启动
```





### 文件



目录

ClickHouse各文件目录：
    bin/      /usr/bin/ 
    conf/     /etc/clickhouse-server/
    lib/     /var/lib/clickhouse 
    log/    /var/log/clickhouse-server







PartitionId_MinBlockNum_MaxBlockNum_Level
分区值 最小分区块编号 最大分区块编号_合并层级

/var/lib/clickhouse/data/default/t_order_mt/20200602_2_2_0    

 数据分区规则由分区ID决定，分区ID由PARTITION BY分区键决定。根据分区键字段类型，ID生成规则可分为：

未定义分区键
	没有定义PARTITION BY，默认生成一个目录名为all的数据分区，所有数据均存放在all目录下。

 整型分区键
 	分区键为整型，那么直接用该整型值的字符串形式做为分区ID。

日期类分区键
	分区键为日期类型，或者可以转化成日期类型。

其他类型分区键
	String、Float类型等，通过128位的Hash算法取其Hash值作为分区ID。

​    

=》MinBlockNum
    最小分区块编号，自增类型，从1开始向上递增。每产生一个新的目录分区就向上递增一个数字。
=》MaxBlockNum
    最大分区块编号，新创建的分区MinBlockNum等于MaxBlockNum的编号。
=》Level
    合并的层级，被合并的次数。合并次数越多，层级值越大。

​     

bin文件：数据文件
mrk文件：标记文件
    标记文件在 idx索引文件 和 bin数据文件 之间起到了桥梁作用。
    以mrk2结尾的文件，表示该表启用了自适应索引间隔。
primary.idx文件：主键索引文件，用于加快查询效率。
minmax_create_time.idx：分区键的最大最小值。
checksums.txt：校验文件，用于校验各个文件的正确性。存放各个文件的size以及hash值。



## data type 数据类型





### 整型

固定长度的整型，包括有符号整型或无符号整型。



Int8 - [-128 : 127]

Int16 - [-32768 : 32767]

Int32 - [-2147483648 : 2147483647]

Int64 - [-9223372036854775808 : 9223372036854775807] 无符号整型范围(0~2n-1):

UInt8 - [0 : 255]

UInt16 - [0 : 65535]

UInt32 - [0 : 4294967295]

UInt64 - [0 : 18446744073709551615]



### 浮点型

Float32 - float

Float64 – double



使用场景:一般数据值比较小，不涉及大量的统计计算，精度要求不高的时候。比如保存商品的重量。





### 布尔型

没有单独的类型来存储布尔值。可以使用 UInt8 类型，取值限制为 0 或 1。



### Decimal

有符号的浮点数，可在加、减和乘法运算过程中保持精度。对于除法，最低有效数字会 被丢弃(不舍入)。



有三种声明:

Decimal32(s)，相当于Decimal(9-s,s)，有效位数为1~9

Decimal64(s)，相当于Decimal(18-s,s)，有效位数为1~18

Decimal128(s)，相当于Decimal(38-s,s)，有效位数为1~38



s 标识小数位

使用场景: 一般金额字段、汇率、利率等字段为了保证小数点精度，都使用 Decimal

进行存储。



### 字符串

String 

字符串可以任意长度的。它可以包含任意的字节集，包含空字节。





FixedString(N)

固定长度 N 的字符串，N 必须是严格的正自然数。当服务端读取长度小于 N 的字符串时候，通过在字符串末尾添加空字节来达到 N 字节长度。 当服务端读取长度大于 N 的 字符串时候，将返回错误消息。

### 枚举



包括 Enum8 和 Enum16 类型。Enum 保存 'string'= integer 的对应关系。 

Enum8 用 'String'= Int8 对描述。

Enum16 用 'String'= Int16 对描述。



```sql
CREATE TABLE t_enum
(
    x Enum8('hello' = 1, 'world' = 2)
)
ENGINE = TinyLog
```



```sql
INSERT INTO t_enum VALUES ('hello'), ('world')

SELECT CAST(x, 'Int8') FROM t_enum;
```





### 时间类型

Date 接受年-月-日的字符串比如‘2019-12-16’

Datetime 接受年-月-日 时:分:秒的字符串比如 ‘2019-12-16 20:50:10’

Datetime64 接受年-月-日 时:分:秒.毫秒的字符串比如‘2019-12-16 20:50:10.66'



## 表引擎







表引擎决定了如何存储表的数据，包括如下

- 数据的存储方式和位置，写到哪里以及从哪里读取数据。
- 支持哪些查询以及如何支持。
- 并发数据访问
- 索引的使用(如果存在)。
- 是否可以执行多线程请求。
- 数据复制参数







### TinyLog

以列文件的形式保存在磁盘上，不支持索引，没有并发控制。一般保存少量数据的小表， 生产环境上作用有限。可以用于平时练习测试用。

```sql
create table t_tinylog ( id String, name String) engine=TinyLog;
```

### Memory

内存引擎，数据以未压缩的原始形式直接保存在内存当中，服务器重启数据就会消失。 读写操作不会相互阻塞，不支持索引。简单查询下有非常非常高的性能表现(超过 10G/s)。





### MergeTree

支持索引和分区



```sql
create table t_order_mt(
   id UInt32,
   sku_id String,
   total_amount Decimal(16,2),
   create_time Datetime
) engine = MergeTree
partition by toYYYYMMDD(create_time) 
primary key (id)
order by (id,sku_id);
```



```sql
insert into t_order_mt 
values 
	(101,'sku_001',1000.00,'2020-06-01 12:00:00') , 
	(102,'sku_002',2000.00,'2020-06-01 11:00:00'), 
	(102,'sku_004',2500.00,'2020-06-01 12:00:00'), 
	(102,'sku_002',2000.00,'2020-06-01 13:00:00'), 
	(102,'sku_002',12000.00,'2020-06-01 13:00:00'), 
	(102,'sku_002',600.00,'2020-06-02 12:00:00');

```



#### partition by 分区

如果不写 则使用一个分区

MergeTree 是以列文件+索引文件+表定义文件组成的，但是如果设定了分区那么这些文件就会保存到不同的分区目录中。

对与不同分区的数据 会进行并行处理



任何一个批次的数据写入都会产生一个临时分区，不会纳入任何一个已有的分区。写入后的某个时刻(大概 10-15 分钟后)，ClickHouse 会自动执行合并操作(等不及也可以手动 通过 optimize 执行)，把临时分区的数据，合并到已有分区中。





![](https://raw.githubusercontent.com/imattdu/img/main/img/202305160100516.png)



```sql
optimize table t_order_mt final;
```





![](https://raw.githubusercontent.com/imattdu/img/main/img/202305160102307.png)





#### primary key 主键



ClickHouse 中的主键，和其他数据库不太一样，它只提供了数据的一级索引，但是却不是唯一约束





index granularity: 直接翻译的话就是索引粒度，指在稀疏索引中两个相邻索引对应数 据的间隔。

ClickHouse 中的 MergeTree 默认是 8192。官方不建议修改这个值，除非该列存在大量重复值，比如在一个分区中几万行才有一个不同数据。





其中 GRANULARITY N 是设定二级索引对于一级索引粒度的粒度。



![](https://raw.githubusercontent.com/imattdu/img/main/img/202306111121782.png)



#### order by

order by 设定了分区内的数据按照哪些字段顺序进行有序保存。



要求:主键必须是 order by 字段的前缀字段。



#### 二级索引



二级索引的功能在 > v20.1.2.4 默认是开启的。



老版本开启

```sql
set allow_experimental_data_skipping_indices=1;
```



```sql
create table t_order_mt2(
	id UInt32,
	sku_id String,
	total_amount Decimal(16,2), 
	create_time Datetime,
  INDEX a total_amount TYPE minmax GRANULARITY 5
 ) engine = MergeTree
partition by toYYYYMMDD(create_time)
primary key (id)
order by (id, sku_id);
```



GRANULARITY N 设定二级索引对于一级索引粒度的粒度。

分割成小块后，将上记SQL的granularity_value数量的小块组合成一个大的块



#### ttl



TTL 即 Time To Live，MergeTree 提供了可以管理数据表或者列的生命周期的功能。





ttl 字段

- SECOND
- MINUTE
- HOUR
-  DAY
- WEEK
- MONTH
- QUARTER
- YEAR

##### 列设置 

```sql
create table t_order_mt3(
	id UInt32,
	sku_id String,
	total_amount Decimal(16,2) TTL create_time+interval 10 SECOND, 
  create_time Datetime
 ) engine =MergeTree
 partition by toYYYYMMDD(create_time)
primary key (id)
order by (id, sku_id);
 
 
insert into t_order_mt3 
values (106,'sku_001',1000.00,'2020-06-12 22:52:30'), 
(107,'sku_002',2000.00,'2020-06-12 22:52:30'), 
(110,'sku_003',600.00,'2020-06-13 12:00:00');


select * from t_order_mt3
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202305162353934.png)



##### 表设置1

```sql
alter table t_order_mt3 MODIFY TTL create_time + INTERVAL 10 SECOND;
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202305162357183.png)







需要是`Date`or`DateTime`数据类型

##### 表设置2

```sql
CREATE TABLE customers (
timestamp DateTime,
name String,
balance Int32,
address String
)
ENGINE = MergeTree
ORDER BY timestamp
TTL timestamp + INTERVAL 12 HOUR
```





![](https://raw.githubusercontent.com/imattdu/img/main/img/202305170005645.png)







![](https://raw.githubusercontent.com/imattdu/img/main/img/202305170006375.png)







![](https://raw.githubusercontent.com/imattdu/img/main/img/202305170007023.png)









### ReplacingMergeTree

ReplacingMergeTree 是 MergeTree 的一个变种，它存储特性完全继承 MergeTree，只是多了一个去重的功能





#### 去重

##### 时机

数据的去重只会在合并的过程中出现



##### 范围

如果表经过了分区，去重只会在分区内部进行去重，不能执行跨分区的去重。





##### case



ReplacingMergeTree() 填入的参数为版本字段，重复数据保留版本字段值最大的。 

如果不填版本字段，默认按照插入顺序保留最后一条。

```sql
create table t_order_rmt(
   id UInt32,
	sku_id String,
	total_amount Decimal(16,2), 
  create_time Datetime
) engine = ReplacingMergeTree(create_time)
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id, sku_id);


insert into t_order_rmt 
values (101,'sku_001',1000.00,'2020-06-01 12:00:00') , 
(102,'sku_002',2000.00,'2020-06-01 11:00:00'), 
(102,'sku_004',2500.00,'2020-06-01 12:00:00'), 
(102,'sku_002',2000.00,'2020-06-01 13:00:00'), 
(102,'sku_002',12000.00,'2020-06-01 13:00:00'), 
(102,'sku_002',600.00,'2020-06-02 12:00:00');


select * from t_order_rmt

OPTIMIZE TABLE t_order_rmt FINAL;
```



结论：

- order by 字段作为唯一键
- 去重不能跨分区
- 合并分区时才会进行去重
- 判定为重复数据，保留版本字段值最大，如果版本字段相同则保留最后插入的





### SummingMergeTree

对于不查询明细，只关心以维度进行汇总聚合结果的场景, ClickHouse 为了这种场景提供了一种能够“预聚合”的引擎 SummingMergeTree



```sql
create table t_order_smt(
   id UInt32,
   sku_id String,
   total_amount Decimal(16,2) ,
   create_time Datetime
) engine = SummingMergeTree(total_amount)
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id,sku_id);
 
insert into t_order_smt
values (101, 'sku_001', 1000.00, '2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 11:00:00'),
(102,'sku_004',2500.00,'2020-06-01 12:00:00'),
(102,'sku_002',2000.00,'2020-06-01 13:00:00'),
(102,'sku_002',12000.00,'2020-06-01 13:00:00'),
(102,'sku_002',600.00,'2020-06-02 12:00:00');


select * from t_order_smt;
```

![](https://raw.githubusercontent.com/imattdu/img/main/img/202305170020002.png)



非聚合字段取最早的一条





结论：

- 以SummingMergeTree()中指定的列作为汇总数据列（汇总数据列可以填写多列，但是必须为数字列；如果不填，以所有非维度且为数字列的字段为汇总数据列)
- 以 order by 的列为准，作为维度列， 其他的列按插入顺序保留第一行
- 不在一个分区的数据不会被聚合





```sql
select total_amount from XXX where province_name = '' and create_date='xxx'
```



```sql
select sum(total_amount) from province_name='' and create_date='xxx'
```

推荐使用下面这种，因为可能会包含一些还没来得及聚合的临时明细





## SQL



### insert

插入表

```sql
insert into [table_name] values(...),(....)
```

表到表的插入

```sql
insert into [table_name] select a,b,c from [table_name_2]
```





### update & delete

update & delete 都是重操作 且不支持事务

“重”的原因主要是每次修改或者删除都会导致放弃目标数据的原有分区，重建新分区。所以尽量做批量的变更，不要进行频繁小数据的操作。



同步执行的部分其实只是进行 新增数据新增分区和并把旧分区打上逻辑上的失效标记。直到触发分区合并的时候，才会删除旧数据释放磁盘空间





==> 更新 ：  插入一条新的数据，   _version + 1 
    =》 查询的时候加上一个过滤条件，  where version最大

==> 删除： _sign,   0表示未删除，1表示已删除， 同时 version + 1
    =》 查询的时候加上一个过滤条件， where  _sign=0 and version最大
    
==> 时间久了，数据膨胀了 ==》 类似合并机制，怎么把过期数据清除掉



update

```sql
alter table t_order_smt 
update total_amount=toDecimal32(2000.00,2) 
where id=102;
```



delete

```sql
alter table t_order_smt delete where sku_id ='sku_001';
```





### query

- 支持子查询
- 支持 CTE(Common Table Expression 公用表表达式 with 子句)
- 支持各种JOIN，但是JOIN操作无法使用缓存，所以即使是两次相同的JOIN语句，ClickHouse 也会视为两条新 SQL
- 窗口函数(官方正在测试中...)
- 不支持自定义函数
- GROUP BY 操作增加了 with rollup\with cube\with total 用来计算小计和总计。



#### group by with

with rollup:从右至左去掉维度进行小计

```sql
select id , sku_id, sum(total_amount) 
from t_order_mt 
group by id,sku_id with rollup;
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202306112316323.png)







with cube : 从右至左去掉维度进行小计，再从左至右去掉维度进行小计

```sql
select id , sku_id,sum(total_amount) 
from t_order_mt 
group by id,sku_id with cube;
```

![](https://raw.githubusercontent.com/imattdu/img/main/img/202306112317031.png)







with totals: 只计算合计

```sql
select id , sku_id,sum(total_amount) 
from t_order_mt 
group by id,sku_id with totals;
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202306112317318.png)





### alter



#### 新增字段

```sql
alter table tableName add column newcolname String after col1;
```

#### 修改字段类型

```sql
alter table tableName modify column newcolname String;
```

#### 删除字段

```sql
alter table tableName drop column newcolname;
```



### 导出数据

```sql
clickhouse-client --query "select * from t_order_mt where
create_time='2020-06-01 12:00:00'" --format CSVWithNames>
/opt/module/data/rs1.csv
```









## 副本





副本的目的主要是保障数据的高可用性，即使一台 ClickHouse 节点宕机，那么也可以从 其他服务器获得相同的数据。



###  副本写入流程



![](https://raw.githubusercontent.com/imattdu/img/main/img/202306112350938.png)





### conf step



1.启动 zookeeper 集群

2.在/etc/clickhouse-server/config.d 目录下创建一个名为 metrika.xml 的配置文件,内容如下

```xml
<?xml version="1.0"?>
<yandex>
    <zookeeper-servers>
        <node index="1">
            <host>hadoop102</host>
            <port>2181</port>
        </node>
        <node index="2">
            <host>hadoop103</host>
            <port>2181</port>
        </node>
        <node index="3">
            <host>hadoop104</host>
            <port>2181</port>
        </node>
    </zookeeper-servers>
</yandex>
```

3.metrika.xml 同步到其他机器 h2 h3



4./etc/clickhouse-server/config.xml 添加如下 ,并同步到h2,h3

```xml
<zookeeper incl="zookeeper-servers" optional="true" />
<include_from>/etc/clickhouse-server/config.d/metrika.xml</include_from>
```

5.重启h1,h2,h3 ck





### 操作

ck 副本仅同步数据 不能同步表的结构

h2

```sql
create table t_order_rep2 (
   id UInt32,
sku_id String,
total_amount Decimal(16,2), create_time Datetime
 ) engine =ReplicatedMergeTree('/clickhouse/table/01/t_order_rep','rep_102')
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id,sku_id);
```

h3

```sql
create table t_order_rep2 (
   id UInt32,
sku_id String,
total_amount Decimal(16,2), create_time Datetime
 ) engine =ReplicatedMergeTree('/clickhouse/table/01/t_order_rep','rep_103')
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id,sku_id);
```



ReplicatedMergeTree 中， 第一个参数是分片的zk_path一般按照:/clickhouse/table/{shard}/{table_name} 的格式

第二个参数是副本名称，相同的分片副本名称不能相同







## 分片集群





副本虽然能够提高数据的可用性，降低丢失风险，但是每台服务器实际上必须容纳全量数据，对数据的横向扩容没有解决。

通过分片把一份完整的数据进行切分，不同的分片分布到不同的节点上，再通过 Distributed 表引擎把数据拼接起来一同使用。

Distributed 表引擎本身不存储数据，通过分布式逻辑表来写入、分发、路由来操作多台节点不同分片的分布式数据。









### 具体配置

#### 3分片2副本共6个节点集群配置

/etc/clickhouse-server/config.d/metrika.xml

注:也可以不创建外部文件，直接在 config.xml 的<remote_servers>中指定

```xml
<yandex> 
  <remote_servers> 
    <gmall_cluster> <!-- 集群名称-->  
      <shard> 
        <!--集群的第一个分片-->  
        <internal_replication>true</internal_replication>  
        <!--该分片的第一个副本-->  
        <replica> 
          <host>hadoop101</host>  
          <port>9000</port> 
        </replica>  
        <!--该分片的第二个副本-->  
        <replica> 
          <host>hadoop102</host>  
          <port>9000</port> 
        </replica> 
      </shard>  
      <shard> 
        <!--集群的第二个分片-->  
        <internal_replication>true</internal_replication>  
        <replica> 
          <!--该分片的第一个副本-->  
          <host>hadoop103</host>  
          <port>9000</port> 
        </replica>  
        <replica> 
          <!--该分片的第二个副本-->  
          <host>hadoop104</host>  
          <port>9000</port> 
        </replica> 
      </shard>  
      <shard> 
        <!--集群的第三个分片-->  
        <internal_replication>true</internal_replication>  
        <replica> 
          <!--该分片的第一个副本-->  
          <host>hadoop105</host>  
          <port>9000</port> 
        </replica>  
        <replica> 
          <!--该分片的第二个副本-->  
          <host>hadoop106</host>  
          <port>9000</port> 
        </replica> 
      </shard> 
    </gmall_cluster> 
  </remote_servers> 
</yandex>

```





#### 3节点集群

##### 集群规划



![](https://raw.githubusercontent.com/imattdu/img/main/img/202306122243237.png)







​	102

```xml
<macros>
    <shard>01</shard>
    <replica>rep_1_1</replica>
</macros>
```

103

```xml
<macros>
    <shard>01</shard>
    <replica>rep_1_2</replica>
</macros>
```

104

```xml
<macros>
    <shard>02</shard>
    <replica>rep_2_1</replica>
</macros>
```



##### 详细配置

hadoop102 的/etc/clickhouse-server/config.d 目录下创建 metrika-shard.xml 文件



```xml
<?xml version="1.0"?>
<yandex>
    <remote_servers>
        <gmall_cluster>
            <!-- 集群名称-->
            <shard>
                <!--集群的第一个分片-->
                <internal_replication>true</internal_replication>
                <replica>
                    <!--该分片的第一个副本-->
                    <host>hadoop102</host>
                    <port>9000</port>
                </replica>
                <replica>
                    <!--该分片的第二个副本-->
                    <host>hadoop103</host>
                    <port>9000</port>
                </replica>
            </shard>
            <shard>
                <!--集群的第二个分片-->
                <internal_replication>true</internal_replication>
                <replica>
                    <!--该分片的第一个副本-->
                    <host>hadoop104</host>
                    <port>9000</port>
                </replica>
            </shard>
        </gmall_cluster>
    </remote_servers>
    <zookeeper-servers>
        <node index="1">
            <host>hadoop102</host>
            <port>2181</port>
        </node>
        <node index="2">
            <host>hadoop103</host>
            <port>2181</port>
        </node>
        <node index="3">
            <host>hadoop104</host>
            <port>2181</port>
        </node>
    </zookeeper-servers>
    <macros>
        <shard>01</shard>
        <!--不同机器放的分片数不一样-->
        <replica>rep_1_1</replica>
        <!--不同机器放的副本数不一样-->
    </macros>
</yandex>
```





###### hadoop102 的 metrika-shard.xml 同步到 103 和 104



###### 修改103,104宏配置



![](https://raw.githubusercontent.com/imattdu/img/main/img/202306122250486.png)







![](https://raw.githubusercontent.com/imattdu/img/main/img/202306122249809.png)





###### 修改/etc/clickhouse-server/config.xml 中导入文件 并同步到103,104



###### 重启三台机器ck





##### 使用

###### 在102 创建如下表 

- 会自动同步103,104
- 集群名字需要和配置文件保持一致
- 分片和副本名称从配置文件的宏定义中获取

```sql
create table st_order_mt on cluster gmall_cluster (
   id UInt32,
	sku_id String,
	total_amount Decimal(16,2), create_time Datetime
 ) engine = ReplicatedMergeTree('/clickhouse/tables/{shard}/st_order_mt','{replica}')
  partition by toYYYYMMDD(create_time)
  primary key (id)
  order by (id,sku_id);
```



###### 102创建分布式表

Distributed(集群名称，库名，本地表名，分片键) 分片键必须是整型数字，所以用 hiveHash 函数转换，也可以 rand()

```sql
create table st_order_mt_all2 on cluster gmall_cluster
(
id UInt32,
  sku_id String,
total_amount Decimal(16,2), create_time Datetime
)engine = Distributed(gmall_cluster,default, st_order_mt,hiveHash(sku_id));
```



###### 102插入数据

```sql
insert into st_order_mt_all2 values
(201,'sku_001',1000.00,'2020-06-01 12:00:00') ,
(202,'sku_002',2000.00,'2020-06-01 12:00:00'),
(203,'sku_004',2500.00,'2020-06-01 12:00:00'),
(204,'sku_002',2000.00,'2020-06-01 12:00:00'),
(205,'sku_003',600.00,'2020-06-02 12:00:00');
```



###### 查询





```sql
SELECT * FROM st_order_mt_all;
```



```sql
select * from st_order_mt;
```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202306122300801.png)





![](https://raw.githubusercontent.com/imattdu/img/main/img/202306122300269.png)



