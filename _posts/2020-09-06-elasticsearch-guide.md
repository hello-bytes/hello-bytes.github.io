---
layout: post
title: "Elasticsearch，从安装到实践"
subtitle: ''
tags:
  - 笔记 
  - Elasticsearch
---

## 基于docker安装运行

想运行一台单结点的ElasticSearch进行测试，基于Docker可能是最方便的了，一句话就可以搞定：

```
docker run -d --name="elasticsearch" -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.2
```

如果Docker环境没有异常，此时ElasticSearch已经就绪，可以在浏览器访问[http://localhost:9200](http://localhost:9200)检查是否正常。

在我的环境中，返回如下内容：

```
{
  "name" : "4ZzIQax",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "N4V53QuMTyuiW3tA7nWz6Q",
  "version" : {
    "number" : "6.3.2",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "053779d",
    "build_date" : "2018-07-20T05:20:23.451332Z",
    "build_snapshot" : false,
    "lucene_version" : "7.3.1",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## 基本概念

### Index 与 Document

ElasticSearch是面向文档型数据库，一条数据就是一个文档，用JSON作为文档序列化的格式。示例如下：

```
{
  "user": "张三",
  "title": "程序员",
  "desc": "开发一部"
}
```

多个文档就组成了索引（Index），索引Elastic 数据管理的顶层单位，和`MySql`里的数据库是同义词。每个 Index （即数据库）的名字必须是小写。

> 同一个索引下的文档，并不要求其JSON格式完全一样，但建议一样以提高查询效率。

### 关于被去掉的Type

ElasticSearch 设计初期，是直接查考了关系型数据库的设计模式，存在了 Type的概念。Type对应MySql这类关系型数据库的`表`。

搜索引擎是基于 Lucene 的，这种 “基因”决定了 Type 是多余的。 Lucene 的全文检索功能之所以快，是因为 倒序索引 的存在。

而这种倒序索引的生成是基于 Index 的，而并非 Type。多个Type 反而会减慢搜索的速度。

为了保持 Elasticsearch “一切为了搜索” 的宗旨，适当的做些改变（去除 Type）也是无可厚非的，也是值得的。


## 常用的HTTP API

### 列出当前的ElasticSearch的所有Index（索引）


```
curl -X GET 'http://localhost:9200/_cat/indices?v'
```

### 创建一个索引

使用`PUT`方法，访问一个索引的根目录，示例如下：

```
curl -X PUT 'localhost:9200/log'
```

服务器返回一个 JSON 对象：

```
{
    "acknowledged":true,
    "shards_acknowledged":true,
    "index":"log"
}
```

### 删除一个索引

使用`DELETE`方法，访问一个索引的根目录，示例如下：

```
curl -X DELETE 'localhost:9200/log'
```

如果成功，服务器返回一个 JSON 对象：

```
{
    "acknowledged":true,
}
```

### 创建一个文档

鉴于在6.x里，官方已经将`Type`置为`Deprecated`，所以按官方建议，创建文档时，统一使用`_doc`这个默认`Type`

官方文档参考：[https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-type-field.html](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-type-field.html)

以上述`log`这个索引示例：

```
curl -XPUT -H "Content-Type: application/json" 'localhost:9200/log/_doc/1?pretty' -d '{"level":"info","ts":"2020-09-03T21:46:48.098+0800","caller":"runtime/proc.go:204","msg":"Success.."}'
```

ElasticSearch返回：

```
{
  "_index" : "log",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
}
```

### 查看文档

```
curl 'localhost:9200/log/_doc/1?pretty=true'
```

根据我们之前的添加记录，服务器返回：

```
{
  "_index" : "log",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : {
    "level" : "info",
    "ts" : "2020-09-03T21:46:48.098+0800",
    "caller" : "runtime/proc.go:204",
    "msg" : "Success.."
  }
}
```

### 删除文档

发出DELETE请求，跟上文档的访问路径即可：

```
curl -X DELETE 'localhost:9200/log/_doc/1'
```

### 搜索

ElasticSearch 的查询非常特别，使用自己的查询语法，要求 GET 请求带有数据体。

查询条件使用[Match 查询](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/query-dsl-match-query.html)，在`match`字段中指定的匹配条件。如下示例是msg字段里面包含"Success"这个词。

除了指定条件外，还可以指定其它查询参数，如：

- `from`表示返回结果的位移
- `size`表示返回结果的每一页的条数

```
curl -H "Content-Type: application/json" 'localhost:9200/log/_doc/_search'  -d '
{
  "query" : { "match" : { "msg" : "Success" }},
  "from": 0,
  "size": 10
}'
```

返回结果如下：

```
{
    "took":49,
    "timed_out":false,
    "_shards":{
        "total":5,
        "successful":5,
        "skipped":0,
        "failed":0
    },
    "hits":{
        "total":1,
        "max_score":0.2876821,
        "hits":[
            {
                "_index":"log",
                "_type":"_doc",
                "_id":"1",
                "_score":0.2876821,
                "_source":{
                    "level":"info",
                    "ts":"2020-09-03T21:46:48.098+0800",
                    "caller":"runtime/proc.go:204",
                    "msg":"Success.."
                }
            }
        ]
    }
}
```