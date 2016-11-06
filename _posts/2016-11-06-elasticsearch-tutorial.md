---
layout: post
title: "elasticsearch 初步学习"
subtitle: "elasticsearch 入门学习"
date: 2016-11-06 12:50:00
author: "seventhking"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - elasticsearch
---

> "初步学习elasticsearch，安装elasticsearch-head和sense这两个插件"

## 基本概念
Relational DB -> Databases -> Table -> Rows      -> Columns  
ElasticSearch -> Indexes   -> Types -> Documents -> Fields  

* 集群(cluster)  

~~~
GET /_cluster/health

response:
{
    "cluster_name":"elasticsearch",
    "status":"green",	<1>
    "timed_out":false,
    "number_of_nodes":1,
    "number_of_data_nodes":1,
    "active_primary_shards":0,
    "active_shards":0,
    "relocating_shards":0,
    "initializing_shards":0,
    "unassigned_shards":0
}
~~~
  status:  
  1.green，所有主要分片和复制分片都可用  
  2.yellow，所有主要分片可用，但不是所有复制分片都可用  
  3.red，不是所有的主要分片都可用  



* 节点(node)：  
  定义：  
  
* 分片(shards)：  
  定义：最小级别“工作单元”，保存了索引中所有数据的一部分，是一个Lucene实例，文档存在分片中，并在分片中被索引。  
  应用程序与索引通信  
  
  主分片：  
  复制分片：主分片的副本，容灾。  

* 文档  
  文档元数据：  
  _index：文档存储的地方  
  _types：文档代表的对象的类  
  _id：文档的唯一标识



-------------------------------------------------------------------------

* 悲观并发控制(Pessimistic concurrency control)
  这在关系型数据库中被广泛的使用，假设冲突的更改经常发生，为了解决冲突我们把访问区块化。典型的例子  
  是在读一行数据前锁定这行，然后确保只有加锁的那个线程可以修改这行数据
  
* 乐观并发控制(Optimistic concurrency control)
  被Elasticsearch使用，假设冲突不经常发生，也不区块化访问，然而，如果在读写过程中数据发生了变化，  
  更新操作将失败。这时候由程序决定在失败后如何解决冲突。实际情况中，可以重新尝试更新，刷新数据(重新读取)  
  或者直接反馈给用户。
  
~~~
只希望文档的_version是1时更新才生效,否则409，conflict
PUT /website/blog/1?version=1
~~~

~~~
指定外部版本号
PUT /website/blog/2?version=5&version_type=external
{...}
响应的_version号码是5
~~~


## 基本操作

### 创建一个新文档
~~~
注意：_index,_type,_id三者唯一确定一个w恩当，所以最简单的方式是使用POST方法让elasticsearch自动
生成唯一 _id
POST /company/employee/
{...}
~~~

~~~
如果想使用自定义的_id，我们必须告诉elasticsearch应该在_index，_type,_id三者都不同时才接受请求
PUT /company/employee/123?op_type=create
{...}
或者
PUT /company/employee/123/_create
{...}
~~~

### 删除文档
~~~
DELETE /company/employee
~~~

### 更新整个文档
~~~
PUT /company/employee/1
{
    "first_name" : "Jane",
    "last_name" : "Smith",
    "age" : 32,
    "about" : "I like to collect rock albums",
    "interests" : ["music"]
}
~~~

### 文档局部更新

### 获取原始JSON文档
~~~
获取原始JSON文档
GET /company/employee/1
可以只返回感兴趣的source字段
GET /company/employee/1?_source=title,text
若只想得到_source字段而不需要其他元数据，可以这样请求
GET /company/employee/1/_source
~~~

### 简单搜索
~~~
GET /company/employee/_search     //默认返回前10个结果
~~~

### 查询字符串
~~~
GET /company/employee/_search?q=last_name:Smith
~~~

### DSL语句查询
~~~
DSL语句查询
GET /company/employee/_search
{
    "query":{
        "match":{
            "last_name":"Smith"
        }
    }
}

添加filter
GET /company/employee/_search
{
    "query":{
        "filtered":{
            "filter":{
                "range":{
                    "age":{"gt":30}       <1>
                }
            },
            "query":{
                "match":{
                    "last_name":"smith"   <2>
                }
            }
        }
    }
}

全文搜索，包含以下字样
GET /company/employee/_search
{
    "query":{
        "match":{
            "about":"rock climbing"
        }
    }
}

短语搜索，确切匹配短语
GET /company/employee/_search
{
    "query":{
        "match_phrase":{
            "about":"rock climbing"
        }
    }
}

高亮我们的搜索
GET /company/employee/_search
{
    "query":{
        "match_phrase":{
            "about":"rock climbing"
            }
    },
    "highlight":{
        "fields":{
            "about":{}
        }
    }
}

分析，聚合，类似group by
GET /company/employee/_search
{
    "aggs":{
        "all_interests":{
            "terms":{"field":"interests"}
            //聚合也允许分级汇总,如下，统计每种兴趣下职员的平均年龄
            "aggs":{
                "avg":{"field":"age"}
            }
        }
    },
    //可以选择加上查询条件
    "query":{
        "match":{
            "last_name":"smith"
        }
    }
}
~~~

### 检查文档是否存在
~~~
HEAD /company/employee/123
~~~



## 
