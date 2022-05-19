

todo

postman





https://macwk.com/soft/paste



官网

https://www.elastic.co/cn/



## 安装

### mac

#### 方式一

下载地址

https://www.elastic.co/cn/downloads/elasticsearch

mac 7.8 （只有x86版本，目前看兼容





解压即可



```sh
tar -zxvf elasticsearch-7.8.0-darwin-x86_64.tar.gz -C ../module
```

启动 **es安装目录/bin目录下**
```sh
./elasticsearch
```





#### 方式二

```sh
brew install elastic/tap/elasticsearch-full


brew install elastic/tap/kibana-full
```



```sh
brew services start elastic/tap/elasticsearch-full

brew services start elastic/tap/kibana-full
```







### 使用



![](https://raw.githubusercontent.com/imattdu/img/main/img/202205170048766.png)





9300 端口为 Elasticsearch 集群间组件的通信端口，9200 端口为浏览器访问的 http 协议 RESTful 端口。





### 问题

1.Elasticsearch 是使用 java 开发的，且 7.8 版本的 ES 需要 JDK 版本 1.8 以上，默认安装 包带有 jdk 环境，如果系统配置 JAVA_HOME，那么使用系统默认的 JDK，如果没有配 置使用自带的 JDK，一般建议使用系统配置的 JDK。



2.双击启动窗口闪退，通过路径访问追踪错误，如果是“空间不足”，请修改 config/jvm.options 配置文件

```sh
# 设置 JVM 初始内存为 1G。此值可以设置与-Xmx 相同，以避免每次垃圾回收完成后 JVM 重新分配内存
# Xms represents the initial size of total heap space
# 设置 JVM 最大可用内存为 1G
# Xmx represents the maximum size of total heap space
-Xms1g
-Xmx1g
```





## 常用操作





### 基础



![](https://raw.githubusercontent.com/imattdu/img/main/img/202205170053206.png)



ES 里的 Index 可以看做一个库，而 Types 相当于表，Documents 则相当于表的行。 

这里 Types 的概念已经被逐渐弱化，Elasticsearch 6.X 中，一个 index 下已经只能包含一个 type，Elasticsearch 7.X 中, Type 的概念已经被删除了。





### 索引

#### 创建索引





```sh
# put

127.0.0.1:9200/t3
```





```sh
{
 "acknowledged"【响应结果】: true, # true 操作成功
 "shards_acknowledged"【分片结果】: true, # 分片操作成功
 "index"【索引名称】: "shopping"
}
# 注意：创建索引库的分片数默认 1 片，在 7.0.0 之前的 Elasticsearch 版本中，默认 5 片
```







#### 查看所有索引

```sh
# get

127.0.0.1:9200/_cat/indices?v
```



```sh
health status index uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   t4    dqc04Q1dSNOR-p69BrAfPg   1   1          4            0      5.7kb          5.7kb
yellow open   t5    krHsnkW9ST-5CYIPK9qj4A   1   1          7            0     10.3kb         10.3kb
yellow open   t1    _9vxsLJ9SKuc5D9lVjOGiA   1   1          1            1      7.3kb          7.3kb
yellow open   t2    7TKFMj8aRDOOTIuFmVpBzw   1   1          1            1      4.7kb          4.7kb
yellow open   t3    3sS8FYGOSfKTN_TN6qZ_9w   1   1          1            1      4.7kb          4.7kb

```



![](https://raw.githubusercontent.com/imattdu/img/main/img/202205170132408.png)





#### 查看某个具体的索引

```sh
# get

127.0.0.1:9200/t4
```



```sh
{
    "t4": {
        "aliases": {},
        "mappings": {
            "properties": {
                "age": {
                    "type": "integer"
                },
                "name": {
                    "type": "text",
                    "index": false
                },
                "sex": {
                    "type": "text",
                    "index": false
                },
                "t1": {
                    "type": "long"
                }
            }
        },
        "settings": {
            "index": {
                "creation_date": "1652002404530",
                "number_of_shards": "1",
                "number_of_replicas": "1",
                "uuid": "dqc04Q1dSNOR-p69BrAfPg",
                "version": {
                    "created": "7080099"
                },
                "provided_name": "t4"
            }
        }
    }
}
```





```sh
{
   "shopping"【索引名】: {
       "aliases"【别名】: {},
       "mappings"【映射】: {},
       "settings"【设置】: {
           "index"【设置 - 索引】: {
           "creation_date"【设置 - 索引 - 创建时间】: "1614265373911",
           "number_of_shards"【设置 - 索引 - 主分片数量】: "1",
           "number_of_replicas"【设置 - 索引 - 副分片数量】: "1",
           "uuid"【设置 - 索引 - 唯一标识】: "eI5wemRERTumxGCc1bAk2A",
           "version"【设置 - 索引 - 版本】: {
           		"created": "7080099"
           },
 					"provided_name"【设置 - 索引 - 名称】: "shopping"
 					}
 			 }
 		}
}
```







#### 删除索引

```sh
# delete

127.0.0.1:9200/t1
```



### 文档



#### 添加文档



```sh
# post

127.0.0.1:9200/t1/_doc/


{
    "age": 19,
    "name": "dd",
    "type": "t1"
}
```

??? note

    ```sh
    {
         "_index"【索引】: "shopping",
         "_type"【类型-文档】: "_doc",
         "_id"【唯一标识】: "Xhsa2ncBlvF_7lxyCE9G", #可以类比为 MySQL 中的主键，随机生成
         "_version"【版本】: 1,
         "result"【结果】: "created", #这里的 create 表示创建成功
         "_shards"【分片】: {
             "total"【分片 - 总数】: 2,
             "successful"【分片 - 成功】: 1,
             "failed"【分片 - 失败】: 0
         },
         "_seq_no": 0,
         "_primary_term": 1
    }
    ```








#### 添加文档指定id



```sh
# post

127.0.0.1:9200/t5/_doc/7


{
    "age": 12,
    "name": "t52",
    "sex": "0"
}
```



#### 根据id查询文档



```sh
# get


127.0.0.1:9200/t3/_doc/2
```





??? note

    ```sh
    {
         "_index"【索引】: "shopping",
         "_type"【文档类型】: "_doc",
         "_id": "1",
         "_version": 2,
         "_seq_no": 2,
         "_primary_term": 2,
         "found"【查询结果】: true, # true 表示查找到，false 表示未查找到
         "_source"【文档源信息】: {
             "title": "华为手机",
             "category": "华为",
             "images": "http://www.gulixueyuan.com/hw.jpg",
             "price": 4999.00
         }
    }
    ```







#### 修改文档



```sh
# post

127.0.0.1:9200/t1/_doc/2


{
    "age": 99,
    "name": "bb2",
    "type": "t2"
}
```



??? note

    ```sh
    {
         "_index": "shopping",
         "_type": "_doc",
         "_id": "1",
         "_version"【版本】: 2,
         "result"【结果】: "updated", # updated 表示数据被更新
         "_shards": {
             "total": 2,
             "successful": 1,
             "failed": 0
         },
         "_seq_no": 2,
         "_primary_term": 2
    }
    ```





#### 更新部分字段



```sh
# post

127.0.0.1:9200/t1/_update/2


{
    "doc": {
        "name": "bb"
    }
}
```



#### 删除文档

```sh
# delete

127.0.0.1:9200/t1/_doc/2
```



??? note

    ```sh
    {
         "_index": "shopping",
         "_type": "_doc",
         "_id": "1",
         "_version"【版本】: 4, #对数据的操作，都会更新版本
         "result"【结果】: "deleted", # deleted 表示数据被标记为删除
         "_shards": {
             "total": 2,
             "successful": 1,
             "failed": 0
         },
         "_seq_no": 4,
         "_primary_term": 2
    }
    ```







#### 根据条件删除文档



```sh
# delete

127.0.0.1:9200/t1/_delete_by_query


{
    "query": {
        "match": {
            "age": 19
        }
    }
}
```







??? note

    ```sh
    {
       "took"【耗时】: 175,
       "timed_out"【是否超时】: false,
       "total"【总数】: 2,
       "deleted"【删除数量】: 2,
       "batches": 1,
       "version_conflicts": 0,
       "noops": 0,
       "retries": {
           "bulk": 0,
           "search": 0
       },
       "throttled_millis": 0,
       "requests_per_second": -1.0,
       "throttled_until_millis": 0,
       "failures": []
    }
    ```

### mapping



#### 创建mapping





字段名：任意填写，下面指定许多属性，例如：title、subtitle、images、price



String 类型，又分两种： text：可分词 keyword：不可分词，数据会作为完整字段进行匹配

Numerical：数值类型，

​		分两类 基本数据类型：long、integer、short、byte、double、float、half_float 

​		浮点数的高精度类型：scaled_float

Date：日期类型

Array：数组类型

Object：对象

```sh
# post


127.0.0.1:9200/t4/_mapping



{
    
    "properties": {
        "t1": {
            "type": "long"
        }
    }
}
```



#### 查看mapping



```sh
# get

127.0.0.1:9200/t9/_mapping
```



#### 创建索引 带mapping



```sh
# put

127.0.0.1:9200/t5


{
    "settings": {},
    "mappings": {
        "properties": {
            "age": {
                "type": "integer",
                "index": true
            },
            "name": {
                "type": "text",
                "index": true
            },
            "sex": {
                "type": "text",
                "index": true
            }
        }
    }
}
```



### 高级查询



#### 查看某个索引下全部文档

"query"：这里的 query 代表一个查询对象，里面可以有不同的查询属性  

"match_all"：查询类型，例如：match_all(代表查询所有)， match，term ， range 等等 

{查询条件}：查询条件会根据类型的不同，写法也有差异

```sh
# get

127.0.0.1:9200/t5/_search


{
    "query": {
        "match_all": {}
    }
}
```





```sh
{
     "took【查询花费时间，单位毫秒】" : 1116,
     "timed_out【是否超时】" : false,
     "_shards【分片信息】" : {
         "total【总数】" : 1,
         "successful【成功】" : 1,
         "skipped【忽略】" : 0,
         "failed【失败】" : 0
     },
     "hits【搜索命中结果】" : {
         "total"【搜索条件匹配的文档总数】: {
             "value"【总命中计数的值】: 3,
             "relation"【计数规则】: "eq" # eq 表示计数准确， gte 表示计数不准确
         },
         "max_score【匹配度分值】" : 1.0,
         "hits【命中结果集合】" : []
     }
}
```





#### 字段匹配查询

单字段

**match 匹配类型查询，会把查询条件进行分词，然后进行查询，多个词条之间是 or 的关系**

```sh
# get

127.0.0.1:9200/t4/_search


{
    "query": {
        "match": {
            // 或的关系
            "age": 1
        }
    }
}
```

多字段匹配



```sh
# get

127.0.0.1:9200/t4/_search


{
    "query": {
        "multi_match": {
            "query": "1",
            "fields": [
                "age",
                "name"
            ]
        }
    }
}
```



#### 关键字精准查询

单关键字

```sh
# get

127.0.0.1:9200/t4/_search

{
    "query": {
        "term": {
            "age": {
                "value": 100
            }
        }
    }
}
```



多关键字精准查询



```sh
# get

127.0.0.1:9200/t4/_search



{
    "query": {
        "terms": {
            "age": [
                1,
                2,
                3
            ]
        }
    }
}
```



#### 指定查询字段

_source

```sh
# get

127.0.0.1:9200/t4/_search


{
    "_source": [
        "age",
        "name"
    ],
    "query": {
        "terms": {
            "age": [
                1
            ]
        }
    }
}
```



include



includes：来指定想要显示的字段 

excludes：来指定不想要显示的字段



```sh
# get

127.0.0.1:9200/t4/_search

{
    "_source": {
        "excludes": [
            "name",
            "age"
        ]
    },
    "query": {
        "terms": {
            "age": [
                1
            ]
        }
    }
}
```





#### 多个条件组合

`bool`把各种其它查询通过`must`（必须 ）、`must_not`（必须不）、`should`（应该）的方 式进行组合



```sh
# get

127.0.0.1:9200/t5/_search


{
    "query": {
        "bool": {
            // "must": [
            //     {
            //         "match": {
            //             "name": "t51"
            //         }
            //     }
            // ],
            // "must_not": [
            //     {
            //         "match": {
            //             "age": 0
            //         }
            //     }
            // ],
            "should": [
                {
                    "match": {
                        "sex": "0"
                    }
                }
            ]
        }
    }
}
```





#### 范围查询

| 操作符 | 说明 |
| :----: | :--: |
|   gt   |  >   |
|  gte   |  >=  |
|   lt   |  <   |
|  lte   |  <=  |





```sh
# get

127.0.0.1:9200/t5/_search


{
    "query": {
        "bool": {
            "should": [
                {
                    "range": {
                        "age": {
                            "gte": 0,
                            "lte": 35
                        }
                    }
                }
            ]
        }
    }
}
```





#### 模糊查询

fuzziness：编辑距离

```sh
# get

127.0.0.1:9200/t5/_search


{
    "query": {
        "fuzzy": {
            "name": {
                "value": "t51",
                // 也可以不写
                "fuzziness": 2
             
            }
        }
    }
}
```



#### 排序

单字段排序

```sh
# get

127.0.0.1:9200/t5/_search


{
    "query": {
        "fuzzy": {
            "name": {
                "value": "t51"
            }
        }
    },
    "sort": [
        {
            "age": {
                "order": "desc"
            }
        }
    ]
}
```

多字段排序



```sh
# get

127.0.0.1:9200/t5/_search


{
    "query": {
        "fuzzy": {
            "name": {
                "value": "t51"
            }
        }
    },
    "sort": [
        {
            "age": {
                "order": "desc"
            }
        },
        {
            "_score": {
                "order": "desc"
            }
        }
    ]
}
```



#### **高亮查询**



pre_tags：前置标签 

 post_tags：后置标签 

 fields：需要高亮的字段 

title：这里声明 title 字段需要高亮，后面可以为这个字段设置特有配置，也可以空

注意：高亮的字段需要在查询的字段中，如下面的name字段

```sh
# get


127.0.0.1:9200/t5/_search



{
    "query": {
        "match": {
            "name": "t51"
        }
    },
    "highlight": {
        "pre_tags": "<font color='red'>",
        "post_tags": "</font>",
        "fields": {
            "name": {}
        }
    }
}
```





#### 分页查询



from：当前页的起始索引，默认从 0 开始。 from = (pageNum - 1) * size 

size：每页显示多少条

```sh
# get

127.0.0.1:9200/t5/_search


{
    "query": {
        "match": {
            "name": "t52"
        }
    },
    // 保证查询字段有
    "highlight": {
        "pre_tags": "<font color='red'>",
        "post_tags": "</font>",
        "fields": {
            "name": {}
        }
    },
    "from": 0,
    "size": 3
}
```







#### 聚合查询

max,min,sum,avg

cardinality:对某个字段的值进行去重之后再取总数

stats:对某个字段一次性返回 count，max，min，avg 和 sum 五个指标

```sh
# get

127.0.0.1:9200/t5/_search

{
    "aggs": {
        "max_age": {
            "max": {
                "field": "age"
            }
        }
    },
    "size": 0
}


```


```sh
# get

127.0.0.1:9200/t5/_search

{
    "aggs": {
        "stats_age": {
            "stats": {
                "field": "age"
            }
        }
    },
    "size": 0
}


```


根据年龄聚合查询

```sh
# get

127.0.0.1:9200/t5/_search

{
    "aggs": {
        "age_groupby": {
            "terms": {
                "field": "age"
            }
        }
    },
    "size": 0
}


```







```json
{
    "took": 25,
    "timed_out": false,
    "_shards": {
        "total": 1,
        "successful": 1,
        "skipped": 0,
        "failed": 0
    },
    "hits": {
        "total": {
            "value": 7,
            "relation": "eq"
        },
        "max_score": null,
        "hits": []
    },
    "aggregations": {
        "age_groupby": {
            "doc_count_error_upper_bound": 0,
            "sum_other_doc_count": 0,
            "buckets": [
                {
                    "key": 12,
                    "doc_count": 6
                },
                {
                    "key": 1,
                    "doc_count": 1
                }
            ]
        }
    }
}
```

