---
layout: post
title: "elasticsearch 初步学习"
subtitle: "elasticsearch 入门学习"
date: 2016-11-06 12:50:00
author: "Deetch"
header-img: "img/home-bg-o.jpg"
catalog: true
tags:
    - elasticsearch
---

> "初步学习elasticsearch，安装elasticsearch-head和sense这两个插件"

## 前言
具体的增删改查语句最好参考官方的最新文档，elasticsearch权威指南版本太古老，尤其是升级到elasticsearch5.0后。


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

### index 参数
|值|解释|
|-|---|
|analyzed|首先分析这个字符串，然后索引。换言之，以全文形式索引此字段|
|not_analyzed|索引这个字段，使之可以被搜索，但是索引内容和指定值一样。不分析此字段|
|no|不索引这个字段。这个字段不能被搜索到|

string类型字段默认值是analyzed

~~~
{
    "tag":{
        "type":"string"
        "index":"not_analyzed"
    }
}
~~~

### 指定分析器

~~~
{
    "tweet":{
        "type":"string",
        "analyzer":"english"
    }
}
~~~


## 分析操作

### 查看类型映射
~~~
GET /INDEX_NAME/_mapping
~~~

### 查看分析器分析文本结果
~~~
GET /INDEX_NAME/_analyze?analyzer=YOUR_ANALYZER&text=YOUR_TEXT(若是中文，需要urlencode)
~~~

### 查看查询时条件的解释
~~~
GET /INDEX_NAME/TYPE_NAME/_validate/query?explain
{
    "query" : {
        "match" : {
            "field" : "YOUR_TEXT(虽然指定GET，但大多数会按POST请求，所以中文情况下一般不需要进行urlencode，视具体情况决定)"
        }
    }
}
~~~
