## 基础命令





### key





| 命令          | 解释                                                         |
| ------------- | ------------------------------------------------------------ |
| keys *        | 查看当前库所有key  (匹配：keys *1)                           |
| exists key    | 判断某个key是否存在                                          |
| type key      | 查看你的key是什么类型                                        |
| del key       | 删除指定的key数据                                            |
| unlink key    | 根据value选择非阻塞删除，仅将keys从keyspace元数据中删除，真正的删除会在后续异步操作。 |
| expire key 10 | 为给定的key设置过期时间，10s                                 |
| ttl key       | 查看还有多少秒过期，-1表示永不过期，-2表示已过期             |
| select 2      | select命令切换数据库                                         |
| dbsize        | 查看当前数据库的key的数量                                    |
| flushdb       | 清空当前库                                                   |
| flushall      | 通杀全部库                                                   |



### string



最大支持512mb





| 命令                          | 解释                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| set k1 v1                     |                                                              |
| get  k1                       | 查询对应键值                                                 |
| append k1 v1                  | 将给定的v1 追加到原值的末尾                                  |
| strlen k1                     | 获得值的长度                                                 |
| setnx  k1 v1                  | 只有在 key 不存在时  设置 key 的值                           |
| incr k1                       | 将 key 中储存的数字值增1,只能对数字值操作，如果为空，新增值为1 |
| decr k1                       | 将 key 中储存的数字值减1;只能对数字值操作，如果为空，新增值为-1 |
| incrby / decrby k2 <步长>     | 将 key 中储存的数字值增减。自定义步长。                      |
| mset k1 v1 v2 v3 ....         | 同时设置一个或多个 key-value对                               |
| mget  k1 k2 k3                | 同时获取一个或多个 value                                     |
| msetnx k1 v1 k2 v2 k3 v3 .... | 同时设置一个或多个 key-value 对，当且仅当所有给定 key 都不存在。 |
| getrange k1 0 1               | 获得值的范围，类似java中的substring，**前包，后包**          |
| setrange k1 0 1               | 用 <value> 覆写<key>所储存的字符串值，从<起始位置>开始(**索引从0****开始**)。 |
| setnx k1 2 v1                 | 设置键值的同时，设置过期时间，单位秒。                       |
| getset  k1 v2                 | 以新换旧，设置了新值同时获得旧值                             |
|                               |                                                              |



![](https://raw.githubusercontent.com/imattdu/img/main/img/202205221641515.png)



set k1 v1

NX：当数据库中key不存在时，可以将key-value添加数据库

XX：当数据库中key存在时，可以将key-value添加数据库，与NX参数互斥

EX：key的超时秒数

PX：key的超时毫秒数，与EX互斥







### list





| 命令                         | 解释                                            |
| ---------------------------- | ----------------------------------------------- |
| lpush/rpush k1 v1 v2 v3 .... | 从左边/右边插入一个或多个值。                   |
| lpop/rpop k1                 | 从左边/右边吐出一个值。值在键在，值光键亡。     |
| rpoplpush k1 k2              | 从k1列表右边吐出一个值，插到k2列表左边。        |
| lrange k1 0 2                | 按照索引下标获得元素(从左到右)                  |
| lrange mylist 0 -1           | 0左边第一个，-1右边第一个，（0 -1表示获取所有） |
| lindex k1 0                  | 按照索引下标获得元素(从左到右)                  |
| llen k1                      | 获得列表长度                                    |
| linsert k1 before v1 nv1     | 在v1的后面插入nv1插入值                         |
| lrem k1 n v1                 | 从左边删除n个v1(从左到右)                       |
| lset k1 index 122            | 将列表key下标为index的值替换成122               |







### set

相对于list 该集合不重复





| 命令                   | 解释                                                         |
| ---------------------- | ------------------------------------------------------------ |
| sadd k1 v1 v2 v3 v4... | 将一个或多个 member 元素加入到集合 key 中，已经存在的 member 元素将被忽略 |
| smembers k1            | 取出该集合的所有值。                                         |
| sismember k1 v1        | 判断集合k1是否为含有该v1值，有1，没有0                       |
| scard k1               | 返回该集合的元素个数。                                       |
| srem k1 v1 v2 ....     | 删除集合中的某个元素。                                       |
| spop k1                | 随机从该集合中吐出一个值。                                   |
| srandmember k1 n       | 随机从该集合中取出n个值。不会从集合中删除 。                 |
| smove sk1 dk1 v1       | 把集合中一个值从一个集合移动到另一个集合sk1中的v1移动到dk1   |
| sinter k1 k2           | 返回两个集合的交集元素。                                     |
| sunion k1 k2           | 返回两个集合的并集元素。                                     |
| sdiff k1 k2            | 返回两个集合的**差集**元素(key1中的，不包含key2中的)         |

















### hash



| 命令                           | 解释                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| hset k1 f1 v1                  | 给k1集合中的 f1键赋值v1                                      |
| hget k1 f1                     | 从k1集合f1取出 value                                         |
| hmset k1 f1 v1 f2 v2 f3 v3.... | 批量设置hash的值                                             |
| hexists k1 f1                  | 查看哈希表 key 中，给定域 field 是否存在。                   |
| hkeys  k1                      | 列出该hash集合的所有field                                    |
| hvals k1                       | 列出该hash集合的所有value                                    |
| hincrby k1 f1  <increment>     | 为哈希表 key 中的域 field 的值加上增量 1  -1                 |
| hsetnx k1  f1 v1               | 将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 . |
|                                |                                                              |



 













### zset

有分数

| 命令                                                         | 解释                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| zadd k1  100 v1 200 v2 20 v3 .....                           | 将一个或多个 member 元素及其 score 值加入到有序集 key 当中。 |
| zrange  k1 0 100  [WITHSCORES]                               | 返回有序集 key 中，下标在<start><stop>之间的元素；带WITHSCORES，可以让分数一起和值返回到结果集。 |
| zrangebyscore key minmax [withscores] [limit offset count]   |                                                              |
| zrangebyscore z1 0 100 withscores limit 0 100                | 返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于 min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列 |
| zrevrangebyscore key maxmin [withscores] [limit offset count] | 同上，改为从大到小排列。                                     |
| zincrby k1 <increment><value>                                | 为元素的score加上增量                                        |
| zrem k1 v1                                                   | 删除该集合下，指定值的元素                                   |
| zcount k1 <min><max>                                         | 统计该集合，分数区间内的元素个数                             |
| zrank k1 v1                                                  | 返回该值在集合中的排名，从0开始。                            |







关系型数据库

业务逻辑存储有关系的数据





行数据库：把几条数据存在一行

