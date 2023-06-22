## Explain 执行计划





version >= 21.7.3.14

### 基本语法

```sql
EXPLAIN [AST | SYNTAX | PLAN | PIPELINE] [setting = value, ...]
SELECT ... [FORMAT ...]
```



- PLAN:用于查看执行计划，默认值。
	- header 打印计划中各个步骤的 head 说明，默认关闭，默认值 0
	- description 打印计划中各个步骤的概述，默认开启，默认值 1
	- actions 打印计划中各个步骤的详细信息，默认关闭，默认值0
- AST 用于查看语法树
- SYNTAX:用于优化语法
- PIPELINE:用于查看 PIPELINE 计划
	- header 打印计划中各个步骤的 head 说明，默认关闭
	- graph 用DOT图形语言描述管道图，默认关闭，需要查看相关的图形需要配合graphviz 查看
	- actions 如果开启了 graph，紧凑打印打，默认开启



### usage

#### 查看plan

简单查询

```sql
explain plan 
select arrayJoin([1,2,3,null,null]);
```

复杂 SQL 的执行计划

```sql
explain select database,table,count(1) cnt from system.parts where
database in ('datasets','system') group by database,table order by
database,cnt desc limit 2 by database;
```

打开全部的参数的执行计划

```sql
EXPLAIN header=1, actions=1,description=1 SELECT number from
system.numbers limit 10;
```





```sql
explain
select
	database,
	table,
	count(1) cnt
from
	system.parts
where
	database in ('datasets', 'system')
group by
	database,
	table
order by
	database,
	cnt desc
limit 2 by database;
```





![](https://raw.githubusercontent.com/imattdu/img/main/img/202305232230498.png)









![](https://raw.githubusercontent.com/imattdu/img/main/img/202305232233366.png)







#### AST 语法树

```sql
EXPLAIN AST SELECT number from system.numbers limit 10;
```



#### SYNTAX 语法优化

```sql
//先做一次查询
SELECT number = 1 ? 'hello' : (number = 2 ? 'world' : 'atguigu') FROM numbers(10);
//查看语法优化
EXPLAIN SYNTAX SELECT number = 1 ? 'hello' : (number = 2 ? 'world' : 'atguigu') FROM numbers(10);
//开启三元运算符优化
SET optimize_if_chain_to_multiif = 1; //再次查看语法优化
EXPLAIN SYNTAX SELECT number = 1 ? 'hello' : (number = 2 ? 'world' :
'atguigu') FROM numbers(10);
//返回优化后的语句
SELECT multiIf(number = 1, \'hello\', number = 2, \'world\', \'xyz\') FROM numbers(10)
```



#### 查看 PIPELINE

```sql
EXPLAIN PIPELINE SELECT sum(number) FROM numbers_mt(100000) GROUP BY
number % 20;
//打开其他参数
EXPLAIN PIPELINE header=1,graph=1 SELECT sum(number) FROM numbers_mt(10000) GROUP BY number%20;
```





## 建表优化

### data type 数据类型



#### 时间字段

日期时间字段使用日期时间，不要使用字符串（字符串还需要进行转换）



```sql
create table t_type2(
  id UInt32, 
  sku_id String, 
  total_amount Decimal(16, 2), 
  create_time Int32
) engine = ReplacingMergeTree(create_time) partition by toYYYYMMDD(
  toDate(create_time)
) – - 需要转换一次，否则报错 primary key (id) 
order by 
  (id, sku_id);
```

#### 空值存储类型

Nullable： 需要创建一个额外的文件来存储 NULL 的标记，并且 Nullable 列无法被索引



使用字段默认值表示空，或者自行指定一个在业务中无意义的值



```sql
CREATE TABLE t_null(x Int8, y Nullable(Int8)) ENGINE TinyLog;
INSERT INTO t_null VALUES (1, NULL), (2, 3);
SELECT x + y FROM t_null;
```

![](https://raw.githubusercontent.com/imattdu/img/main/img/202306150101434.png)







### 分区索引

- 分区不宜太粗或太细， 一亿条数据推荐(10-30个)
- order by字段即为索引：查询频率大的在前原则；基数特别大的不适合做索引



### 表参数

- Index_granularity 是用来控制索引粒度的，默认是 8192，如非必须不建议调整
- 需要指定ttl



### 写入删除优化

- 尽量不要执行单条或小批量删除和插入操作，这样会产生小分区文件，给后台 Merge 任务带来巨大压力
- 不要一次写入太多分区，或数据写入太快，数据写入太快会导致 Merge 速度跟不上而报错，一般建议每秒钟发起 2-3 次写入操作，每次操作写入 2w~5w 条数据(依服务器 性能而定)





### 常见配置

config.xml 或 users.xml





#### cpu conf

| conf                                       | desc                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| background_pool_size                       | 后台线程池的大小，merge 线程就是在该线程池中执行，该线程池 不仅仅是给 merge 线程用的，默认值 16，允许的前提下建议改成 c pu 个数的 2 倍(线程数)。 |
| background_schedule_pool_size              | 执行后台任务(复制表、Kafka 流、DNS 缓存更新)的线程数。默 认 128，建议改成 cpu 个数的 2 倍(线程数)。 |
| background_distributed_schedule_ pool_size | 设置为分布式发送执行后台任务的线程数，默认 16，建议改成 cpu 个数的 2 倍(线程数)。 |
| max_concurrent_queries                     | 最大并发处理的请求数(包含 select,insert 等)，默认值 100，推荐 1 50(不够再加)~300。 |
| max_threads                                | 设置单个查询所能使用的最大 cpu 个数，默认是 cpu 核数         |
|                                            |                                                              |



#### mem conf

| conf                                | desc                                                         |
| ----------------------------------- | ------------------------------------------------------------ |
| max_memory_usage                    | 此参数在 users.xml 中,表示单次 Query 占用内存最大值，该值可 以设置的比较大，这样可以提升集群查询的上限。 |
| max_bytes_before_external_group_ by | 一般按照 max_memory_usage 的一半设置内存，当 group 使用内存超过阈值后会刷新到磁盘进行。 |
| max_bytes_before_external_sort      | 当 order by 已使用 max_bytes_before_external_sort 内存就进行 溢写磁盘(基于磁盘排序)，如果不设置该值，那么当内存不够时直接 抛错，设置了该值 order by 可以正常完成，但是速度相对存内存来 说肯定要慢点(实测慢的非常多，无法接受)。 |
| max_table_size_to_drop              | 此参数在 config.xml 中，应用于需要删除表或分区的情况，默认是 50GB，意思是如果删除 50GB 以上的分区表会失败。建议修改为 0， 这样不管多大的分区表都可以删除。 |
|                                     |                                                              |



#### 存储

ClickHouse 不支持设置多数据目录，为了提升数据 io 性能，可以挂载虚拟券组，一个券组绑定多块物理磁盘提升读写性能，多数据查询场景 SSD 会比普通机械硬盘快 2-3 倍。





## 语法优化规则



### count 优化



count() 或者 count(*)，且没有 where 条件，则会直接使用 system.tables 的 total_rows

```sql
EXPLAIN SELECT count()FROM datasets.hits_v1;
Union
  Expression (Projection)
   Expression (Before ORDER BY and SELECT)
     MergingAggregated
       ReadNothing (Optimized trivial count)
```



```sql
EXPLAIN SELECT count(CounterID) FROM datasets.hits_v1;
Union
  Expression (Projection)
   Expression (Before ORDER BY and SELECT)
     Aggregating
       Expression (Before GROUP BY)
        ReadFromStorage (Read from MergeTree)
```



### 消除子查询重复字段



```sql
EXPLAIN SYNTAXSELECT a.UserID,
         b.VisitID,
         a.URL,
         b.UserID
FROM hits_v1 AS a
LEFT JOIN 
    (SELECT UserID,
         UserID AS HaHa,
         VisitID
    FROM visits_v1) AS b
USING (UserID) limit 3;
```



```sql
SELECT UserID,
         VisitID,
         URL,
         b.UserID
FROM hits_v1 AS a ALL
LEFT JOIN 
    (SELECT UserID,
         VisitID
    FROM visits_v1 ) AS b
USING (UserID) LIMIT 3
```



### 谓词下推

group by 有having 子句， 但是没有with cube、with rollup 或者 with totals 修饰的时 候，having 过滤会下推到 where 前过滤



```sql
EXPLAIN SYNTAX SELECT UserID FROM hits_v1 GROUP BY UserID HAVING UserID =
'8585742290196126178';
//返回优化语句
SELECT UserID
FROM hits_v1
WHERE UserID = \'8585742290196126178\' GROUP BY UserID
```





```sql
EXPLAIN SYNTAX
SELECT *
FROM
(
   SELECT UserID
   FROM visits_v1
)
WHERE UserID = '8585742290196126178'
//返回优化后的语句 
SELECT UserID FROM
(
   SELECT UserID
   FROM visits_v1
		WHERE UserID = \'8585742290196126178\'
)
WHERE UserID = \'8585742290196126178\'
```





### 聚合计算外推



```sql
EXPLAIN SYNTAX
SELECT sum(UserID * 2)
FROM visits_v1

//返回优化后的语句 
SELECT sum(UserID) * 2
FROM visits_v1
```



### 聚合函数消除

group by 字段使用聚合函数，则函数会消除

```sql
EXPLAIN SYNTAX
SELECT
   sum(UserID * 2),
   max(VisitID),
   max(UserID)
FROM visits_v1
GROUP BY UserID
```



优化后

```sql
SELECT
   sum(UserID) * 2,
   max(VisitID),
UserID
FROM visits_v1
GROUP BY UserID
```

### 删除重复的order by 字段

```sql
EXPLAIN SYNTAX
SELECT *
FROM visits_v1
ORDER BY
   UserID ASC,
   UserID ASC,
   VisitID ASC,
   VisitID ASC
//返回优化后的语句: select
    ......
FROM visits_v1
ORDER BY
   UserID ASC,
   VisitID ASC
```



### 删除重复的limit by 

limit by 相同的字段选前几个

```sql
EXPLAIN SYNTAX
SELECT *
FROM visits_v1
LIMIT 3 BY
VisitID,
   VisitID
LIMIT 10

//返回优化后的语句:
select ......
FROM visits_v1
LIMIT 3 BY VisitID
LIMIT 10
```



### 删除重复的using key



```sql
EXPLAIN SYNTAX
SELECT
   a.UserID,
   a.UserID,
   b.VisitID,
   a.URL,
   b.UserID
   FROM hits_v1 AS a
	 LEFT JOIN visits_v1 AS b USING (UserID, UserID)

  //返回优化后的语句: 
  SELECT
   UserID,
   UserID,
   VisitID,
   URL,
   b.UserID
FROM hits_v1 AS a
ALL LEFT JOIN visits_v1 AS b USING (UserID)
```



### 标量替换

如果子查询只返回一行数据，在被引用的时候用标量替换，例如下面语句中的total_disk_usage 字段:

```sql
EXPLAIN SYNTAX
WITH
   (
       SELECT sum(bytes)
       FROM system.parts
       WHERE active
   ) AS total_disk_usage
SELECT
   (sum(bytes) / total_disk_usage) * 100 AS table_disk_usage,
   table
FROM system.parts
GROUP BY table
ORDER BY table_disk_usage DESC
LIMIT 10;

-- 返回优化后的语句:
WITH CAST(0, \'UInt64\') AS total_disk_usage SELECT
   (sum(bytes) / total_disk_usage) * 100 AS table_disk_usage,
   table
FROM system.parts
GROUP BY table
ORDER BY table_disk_usage DESC
LIMIT 10
```





三元运算符优化

如果开启了 optimize_if_chain_to_multiif 参数，三元运算符会被替换成 multiIf 函数





```sql
EXPLAIN SYNTAX
SELECT number = 1 ? 'hello' : (number = 2 ? 'world' : 'atguigu') FROM numbers(10)
settings optimize_if_chain_to_multiif = 1;


-- optmize
SELECT multiIf(number = 1, \'hello\', number = 2, \'world\', \'atguigu\') FROM numbers(10)
SETTINGS optimize_if_chain_to_multiif = 1
```





## 查询优化



### 单表查询

#### prewhere替代where

prewhere 只支持 *MergeTree 族系列引擎的表，首先会读取指定的列数据，来判断数据过滤，等待数据过滤之后再读取 select 声明的列字段来补全其余属性



```sql
-- 关闭 where 自动转 prewhere(默认情况下， where 条件会自动优化成 prewhere) 
set optimize_move_to_prewhere=0;
```



where 不会转换位prewhere场景

⚫ 使用常量表达式

⚫ 使用默认值为alias类型的字段

⚫ 包含了arrayJOIN，globalIn，globalNotIn或者indexHint的查询 

⚫ select查询的列字段和where的谓词相同

⚫ 使用了主键字段





#### 数据采样

采样修饰符只有在 MergeTree engine 表中才有效，且在创建表时需要指定采样策略。

```sql
SELECT Title,count(*) AS PageViews
FROM hits_v1
SAMPLE 0.1 #代表采样 10%的数据,也可以是具体的条数 WHERE CounterID =57
GROUP BY Title
ORDER BY PageViews DESC LIMIT 1000
```



#### 列裁剪和分区裁剪

数据量太大时应避免使用 select * 操作，查询的性能会与查询的字段大小和数量成线性

表换，字段越少，消耗的 io 资源越少，性能就会越高。



```sql
反例:
select * from datasets.hits_v1;
正例:
select WatchID,
   JavaEnable,
   Title,
   GoodEvent,
EventTime,
   EventDate,
   CounterID,
   ClientIP,
   ClientIP6,
   RegionID,
   UserID
from datasets.hits_v1;
```



```sql
select WatchID,
   JavaEnable,
   Title,
   GoodEvent,
   EventTime,
   EventDate,
   CounterID,
   ClientIP,
   ClientIP6,
   RegionID,
   UserID
from datasets.hits_v1
where EventDate='2014-03-23';
```





#### order by 结合where limit



```sql
#正例:
SELECT UserID,Age
FROM hits_v1
WHERE CounterID=57
ORDER BY Age DESC LIMIT 1000

#反例:
SELECT UserID,Age FROM hits_v1 ORDER BY Age DESC
```



#### 避免构建虚拟列

如非必须，不要在结果集上构建虚拟列，虚拟列非常消耗资源浪费性能，可以考虑在前端进行处理，或者在表中构造实际字段进行额外存储。

```sql

-- 正例
SELECT Income,Age,Income/Age as IncRate FROM datasets.hits_v1;
-- 正例:拿到 Income 和 Age 后，考虑在前端进行处理，或者在表中构造实际字段进行额外存储 
SELECT Income,Age FROM datasets.hits_v1;
```

#### uniqCombined 替代 distinct

性能可提升 10 倍以上，uniqCombined 底层采用类似 HyperLogLog 算法实现，能接收 2% 左右的数据误差，可直接使用这种去重方式提升查询性能。Count(distinct )会使用 uniqExact 精确去重。



```sql
反例:
select count(distinct rand()) from hits_v1;
正例:
SELECT uniqCombined(rand()) from datasets.hits_v1
```



#### 其他

##### 查询熔断

为了避免因个别慢查询引起的服务雪崩的问题，除了可以为单个查询设置超时以外，还可以配置周期熔断，在一个查询周期内，如果用户频繁进行慢查询操作超出规定阈值后将无 法继续进行查询操作。



##### 关闭虚拟内存 

物理内存和虚拟内存的数据交换，会导致查询变慢，资源允许的情况下关闭虚拟内存。





##### 配置 join_use_nulls

为每一个账户添加 join_use_nulls 配置，左表中的一条记录在右表中不存在，右表的相应字段会返回该字段相应数据类型的默认值，而不是标准 SQL 中的 Null 值。



##### 批量写入时先排序 

批量写入数据时，必须控制每个批次的数据中涉及到的分区的数量，在写入之前最好对需要导入的数据进行排序。无序的数据或者涉及的分区太多，会导致 ClickHouse 无法及时对 新导入的数据进行合并，从而影响查询性能。



##### cpu

cpu 一般在 50%左右会出现查询波动，达到 70%会出现大范围的查询超时，cpu 是最关键的指标，要非常关注。





### 多表关联



#### premise

创建表

```sql
#创建小表
CREATE TABLE visits_v2
ENGINE = CollapsingMergeTree(Sign)
PARTITION BY toYYYYMM(StartDate)
ORDER BY (CounterID, StartDate, intHash32(UserID), VisitID) SAMPLE BY intHash32(UserID)
SETTINGS index_granularity = 8192
as select * from visits_v1 limit 10000;
#创建 join 结果表:避免控制台疯狂打印数据
CREATE TABLE hits_v2
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID)) SAMPLE BY intHash32(UserID)
SETTINGS index_granularity = 8192
as select * from hits_v1 where 1=0;
```



#### 用 in 代替 join

```sql
insert into hits_v2
select a.* from hits_v1 a where a. CounterID in (select CounterID from
visits_v1);

-- 反例:使用 join
insert into table hits_v2
select a.* from hits_v1 a left join visits_v1 b on a. CounterID=b. CounterID;
```



#### 大小表join

多表 join 时要满足小表在右的原则，右表关联时被加载到内存中与左表进行比较， 

ClickHouse 中无论是 Left join 、Right join 还是 Inner join 永远都是拿着右表中的每一条记录 到左表中查找该记录是否存在，所以右表必须是小表。





#### 注意谓词下推



```sql
Explain syntax
select a.* from hits_v1 a left join visits_v2 b on a. CounterID=b.
CounterID
having a.EventDate = '2014-03-17';

Explain syntax
select a.* from hits_v1 a left join visits_v2 b on a. CounterID=b.
CounterID
having b.StartDate = '2014-03-17';

insert into hits_v2
select a.* from hits_v1 a left join visits_v2 b on a. CounterID=b.
CounterID
where a.EventDate = '2014-03-17';

insert into hits_v2
select a.* from (
   select * from
   hits_v1
   where EventDate = '2014-03-17'
) a left join visits_v2 b on a. CounterID=b. CounterID;
```



#### 分布式表使用 GLOBAL



两张分布式表上的 IN 和 JOIN 之前必须加上 GLOBAL 关键字，

使用global 右表只会在接收到查询请求的节点查询， 否则每个节点都需要查询



#### 使用字典表

将一些需要关联分析的业务创建成字典表进行 join 操作，前提是字典表不宜太大，因为字典表会常驻内存



#### 提前过滤

通过增加逻辑过滤可以减少数据扫描，达到提高执行速度及降低内存消耗的目的





## 数据一致性



### overview

ReplacingMergeTree 和 MergeTree 不同之处在于它会删除排序键值相同的重复项

数据去重只会在数据合并期间进行，也可以通过 OPTIMZE 语句发起计划外的合并。但是会引发大量读写



下面给出解决一致性的解决方案



### premise 前提

#### 创建表

``` sql
CREATE TABLE test_a( user_id UInt64, score String, deleted UInt8 DEFAULT 0, create_time DateTime DEFAULT toDateTime(0) ) ENGINE = ReplacingMergeTree(create_time) 
ORDER BY
   user_id;
```

- user_id 是数据去重更新的标识;
- create_time 是版本号字段，每组数据中 create_time 最大的一行表示最新的数据; 
- deleted 是自定的一个标记位，比如 0 代表未删除，1 代表删除数据。



#### 写入100w 条数据

```sql
INSERT INTO
   TABLE test_a(user_id, score) WITH ( 
   SELECT
      [ 'A', 'B', 'C', 'D', 'E', 'F', 'G' ] ) AS dictSELECT number AS user_id,
      dict[number % 7 + 1] 
   FROM
      numbers(10000000);

```



#### 修改前50w行数据

```sql
INSERT INTO
   TABLE test_a(user_id, score, create_time) WITH ( 
   SELECT
      [ 'AA', 'BB', 'CC', 'DD', 'EE', 'FF', 'GG' ] ) AS dictSELECT number AS user_id,
      dict[number % 7 + 1],
      now() AS create_time 
   FROM
      numbers(500000);
```

#### 统计总数

```sql
SELECT COUNT() FROM test_a;

-- output 10500000
```



### 具体方案

#### 手动 OPTIMIZE

```sql
OPTIMIZE TABLE test_a FINAL;
语法:OPTIMIZE TABLE [db.]name [ON CLUSTER cluster] [PARTITION partition | PARTITION ID 'partition_id'] [FINAL] [DEDUPLICATE [BY expression]]
```





#### GROUP BY 去重



```sql
SELECT
user_id ,
  argMax(score, create_time) AS score,
  argMax(deleted, create_time) AS deleted,
  max(create_time) AS ctime
FROM test_a
GROUP BY user_id
HAVING deleted = 0;
```



argMax(field1，field2):按照 field2 的最大值取 field1 的值。



##### 测试

创建视图

```sql
CREATE VIEW view_test_a AS
SELECT
  user_id ,
  argMax(score, create_time) AS score,
  argMax(deleted, create_time) AS deleted,
  max(create_time) AS ctime
FROM test_a
GROUP BY user_id
HAVING deleted = 0;
```

插入重复数据，再次查询

```sql
#再次插入一条数据
INSERT INTO TABLE test_a(user_id,score,create_time) VALUES(0,'AAAA',now())
#再次查询
SELECT *
FROM view_test_a WHERE user_id = 0;
```

删除数据测试



```sql
#再次插入一条标记为删除的数据
INSERT INTO TABLE test_a(user_id,score,deleted,create_time) VALUES(0,'AAAA',1,now());
#再次查询，刚才那条数据看不到了 
SELECT *
FROM view_test_a
WHERE user_id = 0;
```

#### FINAL 查询



早期版本 FINAL 会将查询变为单线程

v20.5.2.7-stable 版本中，FINAL 查询支持多线程执行，并且可以通过 max_final_threads 参数控制单个查询的线程数





普通查询

```sql
select * from visits_v1 WHERE StartDate = '2014-03-17' limit 100 settings
max_threads = 2;


explain pipeline select * from visits_v1 WHERE StartDate = '2014-03-17'
limit 100 settings max_threads = 2;
(Expression)
ExpressionTransform × 2
(SettingQuotaAndLimits) (Limit)
Limit 2 → 2
     (ReadFromMergeTree)
     MergeTreeThread × 2 0 → 1
```





FINAL查询

查询速度没有普通的查询快，但是相比之前已经有了一些提升,查看 FINAL 查询的执行计划:

从 CollapsingSortedTransform 这一步开始已经是多线程执行，但是读取 part 部分的动 作还是串行。

```sql
explain pipeline select * from visits_v1 final WHERE StartDate = '2014- 03-17' limit 100 settings max_final_threads = 2;
(Expression)
ExpressionTransform × 2
  (SettingQuotaAndLimits) (Limit)
  Limit 2 → 2
       (ReadFromMergeTree)
       ExpressionTransform × 2
         CollapsingSortedTransform × 2
  					Copy 1 → 2 
  							AddingSelector
                  ExpressionTransform
                   MergeTree 0 → 1
```



## 物化视图

### overview



ClickHouse 的物化视图是一种查询结果的持久化，它确实是给我们带来了查询效率的提升



#### 物化视图和普通视图区别

普通视图不保存数据，保存的仅仅是查询语句

物化视图则是把查询的结果根据相应的引擎存入到了磁盘 或内存中，对数据重新进行了组织



#### 优缺点

优点：

- 查询速度快，要是把物化视图这些规则全部写好，它比原数据查询快了很多，总的行数少了，因为都预计算好了。



缺点：

- 历史数据做去重、去核这样的分析，在物化视图里面是不太好用的
- 如果一张表加了好多物化视图，在写这张表的时候，就会消耗很多机器的资源

### usage

#### 基本语法

```sql
CREATE [MATERIALIZED] VIEW [IF NOT EXISTS] [db.]table_name [TO[db.]name]
[ENGINE = engine] [POPULATE] AS SELECT ...
```

也可以 TO 表名，保存到一张显式的表。没有加 TO 表名，表名默认就是 .inner.物化视图名



##### 限制

- 必须指定物化视图的 engine 用于数据存储
- TO [db].[table]语法的时候，不得使用 POPULATE
- 查询语句(select)可以包含下面的子句: DISTINCT, GROUP BY, ORDER BY, LIMIT... 
- 物化视图的 alter 操作有些限制，操作起来不大方便。
- 若物化视图的定义使用了 TO [db.]name 子语句，则可以将目标表的视图 卸载DETACH 再装载 ATTACH

##### 物化视图数据更新

- 物化视图创建好之后，若源表被写入新数据则物化视图也会同步更新
- POPULATE 关键字决定了物化视图的更新策略:
	- 若有 POPULATE 则在创建视图的过程会将源表已经存在的数据一并导入，类似于 create table ... as
	- 若无 POPULATE 则物化视图在创建之后没有数据，只会在创建只有同步之后写入 源表的数据
	- clickhouse 官方并不推荐使用 POPULATE，因为在创建物化视图的过程中同时写入 的数据不能被插入物化视图。
- 物化视图不支持同步删除，若源表的数据不存在(删除了)则物化视图的数据仍然保留
- 物化视图是一种特殊的数据表，可以用 show tables 查看 



#### special

##### 准备表与数据

创建表

```sql
#建表语句
CREATE TABLE hits_test (
   EventDate Date,
   CounterID UInt32,
   UserID UInt64,
   URL String,
   Income UInt8
)
ENGINE = MergeTree()
PARTITION BY toYYYYMM(EventDate)
ORDER BY (CounterID, EventDate, intHash32(UserID))
SAMPLE BY intHash32(UserID)
SETTINGS index_granularity = 8192
```

导入数据

```sql
INSERT INTO hits_test
   SELECT
   EventDate,
   CounterID,
   UserID,
   URL,
   Income
FROM hits_v1
limit 10000;
```

##### 创建物化视图

```sql
#建表语句
CREATE MATERIALIZED VIEW hits_mv ENGINE = SummingMergeTree PARTITION BY toYYYYMM(EventDate) 
ORDER BY 
  (
    EventDate, 
    intHash32(UserID)
  ) AS 
SELECT 
  UserID, 
  EventDate, 
  count(URL) as ClickCount, 
  sum(Income) AS IncomeSum 
FROM 
  hits_test 
WHERE 
  EventDate >= '2014-03-20' 
  #设置更新点,该时间点之前的数据可以另外通过
  #insert into select ...... 的方式进行插入
GROUP BY 
  UserID, 
  EventDate;

```

##### 导入增量数据

```sql
#导入增量数据
INSERT INTO hits_test 
SELECT 
  EventDate, 
  CounterID, 
  UserID, 
  URL, 
  Income 
FROM 
  hits_v1 
WHERE 
  EventDate >= '2014-03-23' 
limit 
  10;


#查询物化视图
SELECT 
  * 
FROM 
  hits_mv;
```



##### 导入历史数据

```sql
-- 导入增量数据
INSERT INTO hits_mv 
SELECT 
  UserID, 
  EventDate, 
  count(URL) as ClickCount, 
  sum(Income) AS IncomeSum 
FROM 
  hits_test 
WHERE 
  EventDate = '2014-03-20' 
GROUP BY 
  UserID, 
  EventDate 

-- 查询物化视图
SELECT 
  * 
FROM 
  hits_mv;
```









## MaterializeMySQL 引擎



### overview

ClickHouse 20.8.2.3 版本新增加了 MaterializeMySQL 的 database 引擎，该 database 能映射到 MySQL 中的某个 database ，并自动ClickHouse 中创建对应的 ReplacingMergeTree。



ClickHouse 服务做为 MySQL 副本，读取 Binlog 并执行 DDL 和 DML 请求，实现了基于 MySQL Binlog 机制的业务数据库实时同步功能。





#### feature

- MaterializeMySQL 同时支持全量和增量同步，在 database 创建之初会全量同步 MySQL 中的表和数据，之后则会通过 binlog 进行增量同步。
- MaterializeMySQL database 为其所创建的每张 ReplacingMergeTree 自动增加了 _sign 和 _version 字段。



其中，_version 用作 ReplacingMergeTree 的 ver 版本参数，每当监听到 insert、update 和 delete 事件时，在 databse 内全局自增。

而 _sign 则用于标记是否被删除，取值 1 或 者 -1。

支持如下事件

- MYSQL_WRITE_ROWS_EVENT: _sign = 1，_version ++
- MYSQL_DELETE_ROWS_EVENT: _sign = -1，_version ++
- MYSQL_UPDATE_ROWS_EVENT: 新数据 _sign = 1
- MYSQL_QUERY_EVENT: 支持 CREATE TABLE 、DROP TABLE 、RENAME TABLE 等。





#### 使用细则



##### DDL查询

MySQL DDL 查询被转换成相应的 ClickHouse DDL 查询(ALTER, CREATE, DROP, RENAME)。 如果 ClickHouse 不能解析某些 DDL 查询，该查询将被忽略。



##### 数据复制

MaterializeMySQL 不支持直接插入、删除和更新查询，而是将 DDL 语句进行相应转换: MySQL INSERT 查询被转换为 

INSERT with _sign=1。

MySQL DELETE 查询被转换为 INSERT with _sign=-1。

MySQL UPDATE 查询被转换成 INSERT with _sign=1 和 INSERT with _sign=-1。



##### SELECT查询

如果在 SELECT 查询中没有指定_version，则使用 FINAL 修饰符，返回_version 的最大值对应的数据，即最新版本的数据。

如果在 SELECT 查询中没有指定_sign，则默认使用 WHERE _sign=1，即返回未删除状态(_sign=1)的数据。





##### 索引转换

ClickHouse 数据库表会自动将 MySQL 主键和索引子句转换为 ORDER BY 元组。

ClickHouse 只有一个物理顺序，由 ORDER BY 子句决定。如果需要创建新的物理顺序， 请使用物化视图。



### usage



#### MySQL 开启 binlog 和 GTID 模式

1.确保 MySQL 开启了 binlog 功能，且格式为 ROW 

打开/etc/my.cnf,在[mysqld]下添加:

```sql
server-id=1
log-bin=mysql-bin
binlog_format=ROW
```



2.开启 GTID 模式

如果如果 clickhouse 使用的是 20.8 prestable 之后发布的版本，那么 MySQL 还需要配置 开启 GTID 模式, 这种方式在 mysql 主从模式下可以确保数据同步的一致性(主从切换时)。

```sh
gtid-mode=on
enforce-gtid-consistency=1 # 设置为主从强一致性 
log-slave-updates=1 #记录日志
```

GTID 是 MySQL 复制增强版，从 MySQL 5.6 版本开始支持，目前已经是 MySQL 主流 复制模式。它为每个 event 分配一个全局唯一 ID 和序号，我们可以不用关心 MySQL 集群 主从拓扑结构，直接告知 MySQL 这个 GTID 即可。







3.重启mysql

```sh
sudo systemctl restart mysqld
```



#### 准备mysql表和数据



1.在mysql创建表并写入数据

```sql
CREATE DATABASE testck;

CREATE TABLE `testck`.`t_organization` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `code` int NOT NULL,
  `name` text DEFAULT NULL,
  `updatetime` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY (`code`)
) ENGINE=InnoDB;

INSERT INTO testck.t_organization (code, name,updatetime)
VALUES(1000,'Realinsight',NOW());
INSERT INTO testck.t_organization (code, name,updatetime)
VALUES(1001, 'Realindex',NOW());
INSERT INTO testck.t_organization (code, name,updatetime)
VALUES(1002,'EDT',NOW());
```

2.创建第二张表

```sql
CREATE TABLE `testck`.`t_user` (
    `id` int(11) NOT NULL AUTO_INCREMENT,
  `code` int,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
INSERT INTO testck.t_user (code) VALUES(1);
```

#### 开启clickhouse 物化引擎



```sql
set allow_experimental_database_materialize_mysql=1;
```

#### 创建复制管道

1.ClickHouse 中创建 MaterializeMySQL 数据库

```sh
CREATE DATABASE test_binlog ENGINE =
MaterializeMySQL('hadoop1:3306','testck','root','000000');
```

其中 4 个参数分别是 MySQL 地址、databse、username 和 password。





查看clickhouse 数据

```sql
use test_binlog;
show tables;
select * from t_organization;
select * from t_user;
```



#### 修改数据

1.mysql修改数据

```sql
update t_organization set name = CONCAT(name,'-v1') where id = 1
```

2.查询ck

```sql
select * from t_organization;
```

#### 删除数据

1.mysql查询数据

```sql
DELETE FROM t_organization where id = 2;
```

2.查询ck

```sql
select * from t_organization;
```



3.在刚才的查询中增加 _sign 和 _version 虚拟字段

```sql
select *,_sign,_version from t_organization order by _sign
desc,_version desc;
```

在查询时，对于已经被删除的数据，_sign=-1，ClickHouse 会自动重写 SQL，将 _sign = -1 的数据过滤掉;



![](https://raw.githubusercontent.com/imattdu/img/main/img/202306181231346.png)





```sql
select * from t_organization
等同于
select * from t_organization final where _sign = 1
```



#### 删除表

1.mysql删除表

```sql
drop table t_user;
```



2.查询mysql同步的表会报错

```sql
show tables;
select * from t_user;
DB::Exception: Table scene_mms.scene doesn't exist..
```



3.mysql 新建表，clickhouse 可以查询到

```sql
CREATE TABLE `testck`.`t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `code` int,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB;
INSERT INTO testck.t_user (code) VALUES(1);
#ClickHouse 查询
show tables;
select * from t_user;
```



## 常见问题





### 分布式 DDL 某数据节点的副本不执行

question：

使用分布式 ddl 执行命令 create table on cluster xxxx 某个节点上没有创建 表，但是 client 返回正常，查看日志有如下报错

```sql
<Error> xxx.xxx: Retrying createReplica(), because some other replicas
were created at the same time
```

answer

重启该不执行的节点。



### 数据副本表和数据不一致

question:

由于某个数据节点副本异常，导致两数据副本表不一致，某个数据副本缺少表，需要将两个数据副本调整一致。



answer:

在缺少表的数据副本节点上创建缺少的表，创建为本地表，表结构可以在其他数据副本通过 show crete table xxxx 获取。

表结构创建后，clickhouse 会自动从其他副本同步该表数据，验证数据量是否一致即可。





### 副本节点全量恢复

question:

某个数据副本异常无法启动，需要重新搭建副本。



answer:

清空异常副本节点的 metadata 和 data 目录。

从另一个正常副本将 metadata 目录拷贝过来(这一步之后可以启动数据库，但是只有表结构没有数据)。

执行 sudo -u clickhouse touch /data/clickhouse/flags/force_restore_data 启动数据库。





### 数据副本启动缺少zk表

某个数据副本表在 zk 上丢失数据，或者不存在，但是 metadata 元数据里存在，导致启动异常，报错:

```sh
Can’t get data for node /clickhouse/tables/01-
02/xxxxx/xxxxxxx/replicas/xxx/metadata: node doesn’t exist (No node): Cannot attach table xxxxxxx
```



answer:

metadata 中移除该表的结构文件，如果多个表报错都移除

mv metadata/xxxxxx/xxxxxxxx.sql /tmp/

启动数据库

手工创建缺少的表，表结构从其他节点 show create table 获取。 创建后会自动同步数据，验证数据是否一致。





###  ZK table replicas 数据未删除，导致重建表报错

question:

重建表过程中，先使用 drop table xxx on cluster xxx ,各节点在 clickhouse 上 table 已物理删除，但是 zk 里面针对某个 clickhouse 节点的 table meta 信息未被删除(低概率事件)，因 zk 里仍存在该表的 meta 信息，导致再次创建该表 create table xxx on cluster, 该节点无法创建表(其他节点创建表成功)，报错:

Replica /clickhouse/tables/01-03/xxxxxx/xxx/replicas/xxx already exists..





answer:

从其他数据副本 cp 该 table 的 metadata sql 过来.

重启节点。



### ck意外关闭

question:

模拟其中一个节点意外宕机，在大量 insert 数据的情况下，关闭某个节点。



数据写入不受影响、数据查询不受影响、建表 DDL 执行到异常节点会卡住， 报错:

```sh
Code: 159. DB::Exception: Received from localhost:9000. DB::Exception:
Watching task /clickhouse/task_queue/ddl/query-0000565925 is executing
longer than distributed_ddl_task_timeout (=180) seconds. There are 1
unfinished hosts (0 of them are currently active), they are going to
execute the query in background
```



answer:

启动异常节点，期间其他副本写入数据会自动同步过来，其他副本的 建表 DDL 也会同步。





参考：

https://help.aliyun.com/document_detail/162815.html?spm=a2c4g.11186623.6.652.312e79bd17U8IO
