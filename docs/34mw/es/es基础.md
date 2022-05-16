

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



```sh
# get

127.0.0.1:9200/t5/_search


{
    "query": {
        "match_all": {}
    }
}
```



#### 匹配查询



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



#### 多字段匹配



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



#### 多关键字精准查询



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



#### 指定查询字段  include



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
    }
}
```



#### 单字段排序



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



#### 多字段排序



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



#### 高亮查询



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

11
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

