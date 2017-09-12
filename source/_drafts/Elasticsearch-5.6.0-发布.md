---
title: Elasticsearch 5.6.0 发布
date: 2017-09-12 23:15:00
update: 2017-09-12 23:15:00
tags: [Elasticsearch,Elasticsearch 5.6]
categories: [翻译,Elasticsearch]
---
> [原文来源](https://www.elastic.co/blog/elasticsearch-5-6-0-released)
> [译文来源](./)

今天我们很高兴地宣布推出基于 Lucene 6.6.0的 Elasticsearch 5.6.0版本。这是最新的稳定版本，已经可以在 我们的 [Elasticsearch](https://www.elastic.co/cloud) 服务平台 [Elastic Cloud](https://www.elastic.co/cloud) 上部署。

5.x 最新版本如下：

* [Download Elasticsearch 5.6.0](https://www.elastic.co/downloads/elasticsearch)
* [Elasticsearch 5.6.0 release notes](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/release-notes-5.6.0.html)
* [Elasticsearch 5.6 breaking changes](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/breaking-changes-5.6.html)
* [X-Pack 5.6.0 release notes](https://www.elastic.co/guide/en/x-pack/current/xpack-change-list.html)

您可以在以上链接的发行说明中阅读到所有变更之处，但有一些变更值得注意：

## 升级版本

Elasticsearch 5.6.0是一个升级版本。 如果您想要从5.x滚动升级到6.x的，你需要首先将集群升级到5.6.0或更高版本。 除了允许5.x版本滚动升级到6.x的之外，Elasticsearch 5.6.0附带了最新的弃用日志记录，以向您发出警告，指出您正在使用的已弃用的功能，您需要在迁移到6.0之前进行替换。 即使您计划进行完整的集群重启到6.x，我们强烈建议您首先进入5.6.0或更高版本，并查看弃用日志，以确保您的应用程序已准备好进行升级。

除此之外，在[删除站点插件](https://www.elastic.co/blog/elasticsearch-the-server)之后，我们重新考察了我们如何提供额外的升级辅助工具，因为旧的“迁移助手”实际上是一个站点插件，使得该方法不再起作用。 我们希望将UI和任何新的API捆绑在一起，以便您尽可能轻松地体验迁移路径。 过去，我们也提出了一个CLI兼容版本的旧移民助理的请求。 我们虽然长期坚持这一点，但决定最好的道路是将旧的迁移助手功能（甚至更多！）放在[X-Pack](https://www.elastic.co/downloads/x-pack)中，因为这可以捆绑UI（Kibana）和Elasticsearch扩展。 但是，我们希望确保在X-Pack[完全免费的许可证](https://www.elastic.co/subscriptions)中提供此功能，以免您无需为此类型的基本升级帮助付费。

有了新的 **Upgrade Assistant features** ！ Kibana X-Pack 现在包含一个“Cluster Checkup”工具，用于在升级之前向您发出有关需要在集群中进行更改的警告。 它还包括一个“Reindex Helper”，通过升级Security，Watcher和Kibana索引以及在2.x中创建的索引索引，为6.x兼容性准备索引。 这些Kibana UI由Elastisearch X-Pack中的[新的迁移API](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/migration-api.html)支持。 如果您在纯粹的命令行环境中运行和/或由于某种原因无法安装Kibana，则可以直接使用这些API。

再次说明，这些X-Pack迁移工具需要一个试用许可证（首次使用X-Pack时自动生成）或[免费的基本许可证](https://www.elastic.co/subscriptions)。

注意：如果使用X-Pack安全性，6.0版中需要TLS / SSL加密。 如果您目前正在使用无TLS的X-Pack安全性，则无法进行滚动升级到6.0。 需要完整的集群重新启动才能启用TLS，无论是在5.x还是在过渡到6.x。

## Java高级REST客户端

漫长的等待终于结束了！ 在Elasticsearch 5.0中，我们发布了低级别的Java REST客户端，它为其他语言提供与REST客户端相同的弹性连接，但对于Java程序员来说这有些难以使用，因为它只接受并返回JSON Blob，。 [新的高级REST客户端](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/5.6/java-rest-high.html)构建在低级客户端之上，但它接受请求对象并返回最重要的API的响应对象：info，get，index，delete，update，bulk，search，search-scroll ，和 clear-scroll。 这些构建器和辅助器使客户端更容易在IDE中使用。 其他请求仍然可以用低级客户端来处理。

最终目标是使 Java 与其他语言处于同一级别。 高级HTTP客户端将成为Java应用程序与Elasticsearch通信的正式方式，替换Transport Client，Transport Client 将在Elasticsearch 7.0中被删除。

## Typeless Parent/Child

从即将到来的 6.0 版本开始的最大的改变是移除类型。此 5.6.0版本包含连接数据类型 —— 不需要指定父/子关系类型的新方式。虽然您现有的5.x索引将可以在6.x中继续工作，但我们希望使5.x中的存在连接数据类型以用于帮助您升级到6.0的应用程序。

## 搜索可扩展性

针对诸如 `logstash-*`之类的索引模式的搜索可以引用大量的分片。 通常， 这样的查询包括日期范围过滤器，这意味着大多数分片不会包含任何匹配的文档。 在优化工作完成之前，这一版本中我们已经包含了一个优化以及早终止在这些分片上的请求，但这还不够。Imagine a multi-search request containing 10 search requests, each of which target 2,000 shards. That’s 20,000 shard-level search requests which are added to the search threadpool queue. This could easily result in rejections, even though the majority of these requests are very quick.

Previously, Kibana used the` _field_stats` API with a date range filter to figure out which indices might contain matching documents, and then ran the search request against only those indices. We wanted to remove this API because it was much heavier than users expected and open to abuse. Instead, a search request now has a light shard prefiltering phase which is triggered if a search request targets at least 128 shards (by default). These prefilter requests are not added to the search queue and so cannot be rejected because the queue is full. The prefilter request rewrites the query at the shard level and determines whether the query has any chance of matching any documents at all. The full search request is then sent only to those shards which have a chance of matching.

But what if the user actually does want to search all 2,000 shards, or searches all indices by mistake? These wide-ranging requests should not overwhelm the cluster nor get in the way of search requests from other users. In order to solve this, we introduced the ``max_concurrent_shard_requests`` parameter whose default value depends on the number of nodes in the cluster, but which has a fixed upper limit of 256. This may make a single search request that targets many shards slower, but it makes for fairer concurrent searches by many users.

## 结论

请[下载Elasticsearch 5.6.0](https://www.elastic.co/downloads/elasticsearch)，尝试一下，让我们知道你在Twitter（[@elastic](https://twitter.com/elastic)）或我们[论坛](https://discuss.elastic.co/c/elasticsearch)上的想法。 您可以在[GitHub问题页面](https://github.com/elastic/elasticsearch/issues)上报告任何问题。
