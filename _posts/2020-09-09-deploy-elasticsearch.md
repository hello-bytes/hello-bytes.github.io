---
layout: post
title: "把Elasticsearch推向生产环境"
subtitle: ''
tags:
  - 笔记 
  - Elasticsearch
---

之前的"Elasticsearch，从安装到实践"简单提到了基于Docker安装Elasticsearch，一条shell即可以搞定：

```
docker run -d --name="elasticsearch" -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:6.3.2
```

但在生产环境中，还需要解决以下问题：

- 自定义配置
- 数据文件

## 自定义配置

默认配置文件在Docker的中的目录为`/usr/share/elasticsearch/config/elasticsearch.yml`，为了方便我们修改，简单做一个文件映射即可(假定我们的配置文件在`/etc/elasticsearch/elasticsearch.yml`)：

```
docker run -d --name="elasticsearch" -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -v /usr/share/elasticsearch/config/elasticsearch.yml docker.elastic.co/elasticsearch/elasticsearch:6.3.2
```

> 收于使用了`-d`使用`elasticsearch`以daemon的方式在后台运行，并设置了名称`elasticsearch`,所以再次运行这个shell时需要先运行`docker rm elasticsearch`

默认的配置文件内容为：

```
cluster.name: "docker-cluster"
network.host: 0.0.0.0

# minimum_master_nodes need to be explicitly set when bound on a public IP
# set to 1 to allow single node clusters
# Details: https://github.com/elastic/elasticsearch/pull/17288
discovery.zen.minimum_master_nodes: 1
```

- cluster.name：结点名称
- network.host设为0.0.0.0表示可以从本机以外的其它机器访问
- discovery.zen.minimum_master_nodes:关于集群下选取master结点的限制，这里是单结点，唯持即可

## 数据文件

默认数据文件存放在Docker的`/usr/share/elasticsearch/data`，同样可以使用目录映射将期存到Docker之外的本地目录，在上面示例的基础上，修改shell如下（假定数据保存到本地的`/var/data/elasticsearch`）：

```
docker run -d --name="elasticsearch" -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" -v /usr/share/elasticsearch/config/elasticsearch.yml -v /var/data/elasticsearch:/usr/share/elasticsearch/data docker.elastic.co/elasticsearch/elasticsearch:6.3.2
```







