---
title: Elasticsearch 入门，基础概念
tags: [Elasticsearch,翻译]
categories: [翻译, Elasticsearch]
---

### 基础概念

这里有几个概念是 Elasticsearch 的核心。 从一开始就理解这些概念在很大程度上有助于缓解学习过程难度。


#### 准实时 (Near Realtime, NRT)

Elasticsearch 是一个准实时搜索平台。这意味着你从索引一个文档到它可被搜索到只有轻微的时间延迟（通常为1秒）。


#### 集群 (Cluster)

集群是一个或多个节点（服务器）的集合，他们共同保存你的整个数据，并提供跨所有节点的联合索引和搜索功能。集群由唯一的名称标识，默认情况下是“elasticsearch”。这个名字很重要，因为如果一个节点如果被设置为加入集群，那么他只能通过其名称加入一个集群并成为集群的一部分，

确保不要做不同的环境中使用相同的集群名称，否则可能会导致节点加入错误的集群。例如，您可以对开发，分段和生产集群分别使用`logging-dev`，`logging-stage`和`logging-prod` 作为集群名称。

请注意，拥有只有一个节点的集群是可行和能正常运行的。此外，你也可以拥有多个独立的群集，每个群集都有自己独特的群集名称。


#### 节点 (Node)
节点是作为集群一部分的单一服务器，存储数据，并参与集群的索引和搜索功能。就像一个集群一样，一个节点被一个名称标识，默认情况下是一个随机的通用唯一标识符（UUID），它在启动时分配给节点。如果您不想要默认值，您可以定义所需的任何节点名称。此名称对于管理目的很重要，您希望确定网络中哪些服务器对应于Elasticsearch集群中的哪些节点。

可以将节点配置为按集群名称加入特定集群。默认情况下，每个节点都设置为加入名为的集群elasticsearch，这意味着如果您启动了网络上的多个节点，并且假设它们可以互相发现，则它们将自动形成并加入名为的单个集群elasticsearch。

在单个集群中，您可以拥有所需的节点数。此外，如果您的网络上当前没有其他Elasticsearch节点运行，启动单个节点将默认形成一个名为的新的单节点集群elasticsearch。

#### 索引 (Index)
索引就是具有某些相似特征的文档的一个集合。例如，你可以有一个客户数据索引，另外一个产品目录的索引，以及一个订单数据的索引。一个索引有名称（必须全部为小写字母）标志，这个名字用在对其中的文档进行索引，搜索，更新和删除操作是进行定位索引。

在单个集群里，你可以根据需要定义任意多个索引。


#### 类型 (Type)

在索引中，您可以定义一个或多个类型。类型是你的索引的逻辑分类/分区，其语义完全取决于你。 通常，为具有一组公共字段的文档定义了一种类型。 例如，假设您运行一个博客平台，并将所有数据存储在单个索引中。 在此索引中，您可以定义用户数据类型，博客数据类型以及注释数据类型。


#### 文档 (Document)

文档是可以被索引的基本信息单元。例如，您可以拥有一个客户的文档，一个产品的文档，一个订单的文档。 这个文档以 JSON（JavaScript Object Notation）表示，这是一种应用广泛的互联网数据交换格式。

在索引/类型中，您可以存储你想要存储的任意数量的文档。 请注意，虽然文档物理上位于索引中，但实际上文档实际上必须被索引/分配到索引中的某个类型。


#### 分片和复制 (Shards & Replicas)
An index can potentially store a large amount of data that can exceed the hardware limits of a single node. For example, a single index of a billion documents taking up 1TB of disk space may not fit on the disk of a single node or may be too slow to serve search requests from a single node alone.

To solve this problem, Elasticsearch provides the ability to subdivide your index into multiple pieces called shards. When you create an index, you can simply define the number of shards that you want. Each shard is in itself a fully-functional and independent "index" that can be hosted on any node in the cluster.

Sharding is important for two primary reasons:

* It allows you to horizontally split/scale your content volume
* It allows you to distribute and parallelize operations across shards (potentially on multiple nodes) thus increasing performance/throughput

The mechanics of how a shard is distributed and also how its documents are aggregated back into search requests are completely managed by Elasticsearch and is transparent to you as the user.

In a network/cloud environment where failures can be expected anytime, it is very useful and highly recommended to have a failover mechanism in case a shard/node somehow goes offline or disappears for whatever reason. To this end, Elasticsearch allows you to make one or more copies of your index’s shards into what are called replica shards, or replicas for short.

Replication is important for two primary reasons:

* It provides high availability in case a shard/node fails. For this reason, it is important to note that a replica shard is never allocated on the same node as the original/primary shard that it was copied from.
* It allows you to scale out your search volume/throughput since searches can be executed on all replicas in parallel.

To summarize, each index can be split into multiple shards. An index can also be replicated zero (meaning no replicas) or more times. Once replicated, each index will have primary shards (the original shards that were replicated from) and replica shards (the copies of the primary shards). The number of shards and replicas can be defined per index at the time the index is created. After the index is created, you may change the number of replicas dynamically anytime but you cannot change the number of shards after-the-fact.

By default, each index in Elasticsearch is allocated 5 primary shards and 1 replica which means that if you have at least two nodes in your cluster, your index will have 5 primary shards and another 5 replica shards (1 complete replica) for a total of 10 shards per index.

#### 注意
Elasticsearch 的每一个分片都是一个 Lucene 索引。 你在单个 Lucene 索引里所拥有的文档有最大数量值限制。在 [LUCENE-5843](https://issues.apache.org/jira/browse/LUCENE-5843) 里，这个极限值是 `2,147,483,519` (= `Integer.MAX_VALUE - 128`)。你可以使用 [_cat/shards](https://www.elastic.co/guide/en/elasticsearch/reference/5.5/cat-shards.html) API 来监控分片容量。


在了解这些基础概念后，我们可以进入有趣的部分了。。
