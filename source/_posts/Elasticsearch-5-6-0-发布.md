---
title: Elasticsearch 5.6.0 发布
categories:
  - 翻译
  - Elasticsearch
tags:
  - Elasticsearch
  - Elasticsearch 5.6
date: 2017-09-12 23:15:00
update: 2017-09-12 23:15:00
---

> [原文来源](https://www.elastic.co/blog/elasticsearch-5-6-0-released)

今天（2017/09/11）我们很高兴地宣布推出基于 Lucene 6.6.0的 Elasticsearch 5.6.0版本。这是最新的稳定版本，已经可以在 我们的 [Elasticsearch](https://www.elastic.co/cloud) 服务平台 [Elastic Cloud](https://www.elastic.co/cloud) 上部署。

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

针对诸如 `logstash-*`之类的索引模式的搜索可以引用大量的分片。 通常， 这样的查询包括日期范围过滤器，这意味着大多数分片不会包含任何匹配的文档。 在优化工作完成之前，这一版本中我们已经包含了一个优化以及早终止在这些分片上的请求，但这还不够。想象一下，包含10个搜索请求的多搜索请求，每个搜索请求都有2,000个碎片。这就是20,000个分片级的搜索请求，它们都被添加到搜索线程池队列中。这很容易导致搜索拒绝，即使大部分请求非常快。

以前，Kibana使用使用带有日期范围过滤器的 `_field_stats` API来确定那些索引可能包含匹配的文档，然后针对这些索引运行搜索请求。我们想删除这个API，因为它比用户的预期要重得多，可能被滥用。相反，搜索请求现在具有轻薄分片预过滤阶段，如果搜索请求至少指定128个分片（默认情况下），就会被触发。这些前置过滤器请求不会添加到搜索队列中，所以搜索不会因为队列已满而被拒绝。预过滤器请求在分片级别重写查询，并确定查询是否有可能匹配任何文档。完整的搜索请求仅发送到有机会匹配的分片。
但是，如果用户实际上想要搜索所有2,000个分片，或是错误地搜索所有的索引呢？ 这些广泛的请求不应该压倒集群，也不应妨碍其他用户的搜索请求。 为了解决这个问题，我们引入了 `max_concurrent_shard_requests` 参数，其默认值取决于集群中的节点数量，但是固定的上限为256.这可能会使单个搜索请求针对较多的分片，但是它使 由许多用户进行更公平的并发搜索。

## 最后

请[下载Elasticsearch 5.6.0](https://www.elastic.co/downloads/elasticsearch)，尝试一下，让我们知道你在Twitter（[@elastic](https://twitter.com/elastic)）或我们[论坛](https://discuss.elastic.co/c/elasticsearch)上的想法。 您可以在[GitHub问题页面](https://github.com/elastic/elasticsearch/issues)上报告任何问题。
