---
layout: post
title: es基础
date: 2016-07-26
categories: soft
tags: elasticsearch
---



* [java client接入](#java_client)

  

### 是什么 

分布式的实时文件存储
分布式的实时分析搜索引擎
可扩展到上百台服务器，处理pb级数据，通过restful api进行访问，提供多语言client

### 核心概念 
`集群（cluster）：一组具有相同cluster.name的节点的集合，他们协同工作，共享数据并提供故障转移和扩展功能，当然一个节点也可以组成一个集群。

`节点（node）`：一个运行着的Elasticsearch实例，做为用户，我们能够与集群中的任何节点通信，包括主节点。每一个节点都知道文档存在于哪个节点上，它们可以转发请求到相应的节点上。我们访问的节点负责收集各节点返回的数据，最后一起返回给客户端。这一切都由Elasticsearch处理。

`主节点`：集群中一个节点会被选举为主节点(master),它将临时管理集群级别的一些变更，例如新建或删除索引、增加或移除节点等，任何节点都可以成为主节点。我们例子中的集群只有一个节点，所以它会充当主节点的角色

`分片（shard）`：一个分片是一个最小级别“工作单元(worker unit)”,它只是保存了索引中所有数据的一部分，一个分片就是一个lucene实例，并且本身就是一个搜索引擎，我们的文档存储在分片中，并且在分片中被索引，但是我们的应用程序不会直接与它们通信，取而代之的是，直接与索引通信。

`主分片（primary shard）`：你索引中的每个文档属于一个单独的主分片，所以主分片的数量决定了索引最多能存储多少数据。理论上主分片能存储的数据大小是没有限制的，限制取决于你实际的使用情况。

`复制分片（replica shard）`：主分片的一个副本，它可以防止硬件故障导致的数据丢失，同时可以提供读请求，比如搜索或者从别的shard取回文档

`索引（index）`：存储关联数据，类似RDBMS中的库。实际上，索引只是一个用来指向一个或多个分片(shards)的“逻辑命名空间(logical namespace)”

`文档（document）`：

`类型（type）`：

cluster、node 、以及shard的关系大致如下：

![es](/images/soft/elas_cluster.png)

#### 安装

    下载页：https://www.elastic.co/downloads/elasticsearch
    启动 bin/elasticsearch
    curl -X GET http://localhost:9200/

其api dash里有

### java client接入 

#### node client

节点客户端以无数据节点身份加入集群，换言之，它自己不存储任何数据，但是它知道数据在集群中的具体位置，并且能够直接转发请求到对应的节点上

#### transport client
这个更轻量的传输客户端能够发送请求到远程集群。它自己不加入集群，只是简单转发请求给集群中的节点。
两个Java客户端都通过9300端口与集群交互，使用Elasticsearch传输协议(Elasticsearch Transport Protocol)。集群中的节点之间也通过9300端口进行通信。如果此端口未开放，你的节点将不能组成集群。
Java客户端所在的Elasticsearch版本必须与集群中其他节点一致，否则，它们可能互相无法识别。
我们的使用

    <dependency>
        <groupId>org.elasticsearch</groupId>
        <artifactId>elasticsearch</artifactId>
        <version>1.7.0</version>
    </dependency>

### 集成kibana和marvel 

#### kibana

    https://www.elastic.co/downloads/kibana
    bin/kibana
    http://localhost:5601

#### marvel插件

    # Install Marvel into Elasticsearch
    cd /path/to/elasticsearch
    bin/plugin install license
    bin/plugin install marvel-agent

    # Install Marvel into Kibana
    cd /path/to/kibana
    bin/kibana plugin --install elasticsearch/marvel/latest

    # Start Elasticsearch and Kibana
    cd /path/to/elasticsearch
    bin/elasticsearch

    cd /path/to/kibana
    bin/kibana

    # 访问http://localhost:5601/app/marvel 就可以监控节点状态了

#### 说明 

大概长这样：

![es_overview](/images/soft/es_overview.png)

cluster

![cluster](/images/soft/es_cluster.png)

nodes

![nodes](/images/soft/es_nodes.png)

![node](/images/soft/es_node.png)

indices

![indices](/images/soft/es_indices.png)
