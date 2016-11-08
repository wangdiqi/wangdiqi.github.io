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

### 映射、分析、领域特定语言查询
映射(Mapping):数据在每个字段中的解释说明，用于进行字段类型确认，将每个字段匹配为一种确定的  
（string,number,boolean,date等）数据类型  
  
分析(Analysis):全文是如何处理的可以被搜索的，进行全文文本的分词，以建立供搜索用的反向索引  
  
领域特定语言查询(Query DSL):Elasticsearch使用的灵活的、强大的查询语言  
  
~~~
例子，在索引中有12个tweets,只有一个包含2014-09-15，但是我们看下面查询中的total hists。
GET /_search?q=2014 # 12 个结果
GET /_search?q=2014-09-15 # 还是 12 个结果 !
GET /_search?q=date:2014-09-15 # 1 一个结果
GET /_search?q=date:2014 # 0个结果!
~~~

### 确切值(Exact values) vs 全文文本(Full text)


### 倒排索引
elasticsearch使用一种叫做倒排索引(inverted index)的结构来做快速的全文搜索  
倒排索引由在文档中出现的唯一的单词列表，以及对于每个单词在文档中的位置组成

### 分析和分析器
"Set the shape to semi-transparent by calling set_trans(5)"  
* 标准分析器：  
  set,the,shape,to,semi,transparent,by,calling,set_trans,5  
* 简单分析器：非单个字母的文本切分，每个词转为小写  
  set,the,shape,to,semi,transparent,by,calling,set,trans  
* 空格分析器：
  Set,the,shape,to,semi-transparent,by,calling,set_trans(5)
* 语言分析器：english分析器自带一套英语停用词库——像and或the这些与语义无关的通用词  
  set,shape,semi,transpar,call,set_tran,5

### 测试分析器
~~~
GET	/_analyze?analyzer=standard&text=Text to analyze
~~~

### 指定分析器，通过映射(mapping)人工设置这些字段
elasticsearch支持以下简单字段类型：  
  
|类型|表示的数据类型|
|---|------------|
|String|string|
|Whole number|byte, short, integer, long|
|Floating point|float, double|
|Boolean|boolean|
|Date|date|

当你索引一个包含新字段的文档——一个之前没有的字段——Elasticsearch将使用动态映射猜测字段类型，这类型  
来自于JSON的基本数据类型，使用以下规则：  
  
|JSON type|Field type|
|---------|----------|
|Boolean:true or false|"boolean"|
|Whole number:123|"long"|
|Floationg point:123.45|"double"|
|String,valid date:"2014-09-15"|"date"|
|String:"foo bar"|"string"|



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
~~~
局部文档参数doc，没有的字段就是新增，有的字段就是更新
POST /website/blog/1/_update
{
    "doc":{
        "tags":["testing"],
        "views":0
    }
}
~~~

### 使用Groovy脚本局部更新(搜索、排序、聚合、文档更新)  
~~~
将view数量加1
POST /website/blog/1/_update
{
    "script":"ctx._source.views+=1"
}

新增一个标签到tags数组中
POST /website/blog/1/_update
{
    "script":"ctx._source.tags+=new_tag",
    "params":{
        "new_tag":"search"
    }
}

根据文档内容删除文档
POST /website/blog/1/_update
{
    "script":"ctx.op=ctx._source.views==count?'delete':'none'",
    "params":{
        "count":1
    }
}

更新可能不存在的文档，upsert参数定义文档来使其不存在时被创建
POST /website/pageviews/1/_update
{
    "script":"ctx._source.views+=1",
    "upsert":{
        "views":1
    }
}

更新和冲突，retry_on_conflict指定失败重试次数
POST /website/pageviews/1/_update?retry_on_conflict=5	<1>
{
    "script":"ctx._source.views+=1",
    "upsert":{
        "views":0
    }
}
~~~

### 获取原始JSON文档
* note:查询字符串搜索允许任意用户在索引中任何一个字段上运行潜在的慢查询语句,可能  
  暴露私有信息甚至使你的集群瘫痪。  
  
~~~
获取原始JSON文档
GET /company/employee/1
可以只返回感兴趣的source字段
GET /company/employee/1?_source=title,text
若只想得到_source字段而不需要其他元数据，可以这样请求
GET /company/employee/1/_source
~~~

### 简易搜索
~~~
GET /company/employee/_search?q=last_name:Smith
GET /company/employee/_search?q=+last_name:Smith+tweet:mary

"+"前缀表示语句匹配条件必须被满足
"-"前缀表示语句匹配条件必须不被满足
不加前缀表示是可选的
~~~

* name字段
* date晚于2014-09-10
* _all字段包含"aggregations" 或 "geo"

~~~
?q=+name:(mary john)+date:>2014-09-10+(aggregations geo)
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

### 检索多个文档
~~~
POST /_mget
{
    "docs":[{
        "_index"	:	"website",
        "_type"	:		"blog",
        "_id"	:				2
        },
        {
        "_index"	:	"website",
        "_type"	:		"pageviews",
        "_id"	:				1,
        "_source":	"views"
        }
    ]
}

若所有文档具有相同_index和_type，可以通过简单的ids数组来代替完整的docs数组
POST /website/blog/_mget
{
    "ids":["2","1"]
}
~~~

### 批量操作
bulk API允许我们使用单一请求来实现多个文档的create、index、update、delete  

~~~
bulk请求体如下，请注意"\n",每一行的数据不能包含未被转义的换行符
{action:{metadata}}\n
{request body}\n
{action:{metadata}}\n
{request body}\n
~~~

| action | explain  |
|--------|----------|
| create |当文档不存在时创建|
| index  |创建新文档或替换已有文档|
| update |局部更新文档|
| delete |删除一个文档|

~~~
请求体由文档的_source组成
{"delete":{"_index":"website","_type":"blog","_id":"123"}}\n
{"create":{"_index":"website","_type":"blog","_id":"123"}}\n
{"title":"My first blog post"}\n
{"index":{"_index":"website","_type":"blog"}}\n
{"title":"My second blog post"}\n
POST /_bulk
{"delete":{"_index":"website","_type":"blog","_id":"123"}}\n   <1>
{"create":{"_index":"website","_type":"blog","_id":"123"}}\n
{"title":"My first blog post"}\n
{"index":{"_index":"website","_type":"blog"}}\n
{"title":"My second blog post"}\n
{"update":{"_index":"website","_type":"blog","_id":"123","_retry_on_conflict":"5"}}\n
{"doc":{"title":"My	updated	blog post"}}\n  <2>
~~~

### 多索引和多类别
/_search  
在所有索引的所有类型中搜索  
/gb/_search  
在索引gb的所有类型中搜索  
/gb,us/_search  
在索引gb和us的所有类型中搜索  
/g*,u*/_search  
在以g或u开头的索引的所有类型中搜索  
/gb/user/_search  
在索引gb的类型user中搜索  
/gb,us/user,tweet/_search  
在索引gb和us的类型为user和tweet中搜索  
/_all/user,tweet/_search  
在所有索引的user和tweet中搜索search types user and tweet in all indices  

### 分页
Elasticsearch接受from和size参数:  
size:结果数,默认10  
from:跳过开始的结果数,默认0  
GET /_search?size=5  
GET /_search?size=5&from=5  
GET /_search?size=5&from=10  
  
note：应当避免深度分页！因为分布式系统存在排序问题，会导致极大的浪费，返回不可多于1000个结果的原因  


## 内部过程
Node1  
Node2  
Node3  
  
replication:  
consistency:  
timeout:  
routing:  
  
### 新建、索引和删除文档
write操作，必须在主分片上成功完成才能复制到相关的复制分片上  
  
1.客户端给Node1发送新建、索引或删除请求  
2.节点使用文档的_id确定文档属于分片0.它转发请求到Node3，分片0位于这个节点上  
3.Node3在主分片上执行请求，如果成功，它转发请求到相应的位于Node1和Node2的复制节点上。  
当所有的复制节点报告成功，Node3报告成功返回给请求的节点(Node1)，请求的节点再报告给客户端。  

### 检索文档
read操作  
文档能够从主分片或任意一个复制分片被检索  
对于读请求，为了负载均衡，请求节点会为每个请求选择不同的分片——它会循环所有分片副本  
  
1.客户端给Node1发送GET请求  
2.节点使用文档的_id确定文档属于分片0。分片0对应的复制分片在三个节点上都有。此时，它转发请求到Node2.  
3.Node2返回endangered给Node1然后返回给客户端  

可能的情况有，一个被索引的文档已经存在于主分片上却还没来得及同不到复制分片上。这时复制分片会报告文档未  
杂货哦到，主分片会成功返回文档。*一旦索引请求成功返回给用户，文档则在主分片和复制分片都是可用的。*

### 局部更新文档
结合了之前提到的读和写的模式  
  
1.客户端给Node1发送更新请求  
2.它转发请求到主分片所在节点Node3  
3.Node3从主分片检索出文档，修改_source字段的JSON，然后在主分片上重建索引。如果有其他进程修改了文档，  
它以retry_on_conflict设置的次数重复步骤3，都未成功则放弃
4.如果Node3成功更新文档，它同时转发文档的新版本到Node1和Node2上的复制节点以重建索引。当所有复制节点  
报告成功，Node3返回成功给请求节点，然后返回给客户端

### 多文档模式
mget和bulk API与单独的文档类似。差别是请求节点知道每个文档所在的分片，它把多文档请求拆成每个分片的对  
文档请求，然后转发每个参与的节点。  
  
1.客户端向Node1发送mget请求  
2.Node1为每个分片构建一个多条数据检索请求,然后转发到这些请求所需的主分片或复  
制分片上。当所有回复被接收,Node1构建响应并返回给客户端。  
  
下面我们将罗列使用一个bulk执行多个create、index、delete和update请求的顺序步骤:  
1.客户端向Node1发送bulk请求。  
2.Node1为每个分片构建批量请求,然后转发到这些请求所需的主分片上。  
3.主分片一个接一个的按序执行操作。当一个操作执行完,主分片转发新文档(或者删除  
部分)给对应的复制节点,然后执行下一个操作。一旦所有复制节点报告所有操作已成  
功完成,节点就报告success给请求节点,后者(请求节点)整理响应并返回给客户端。  
