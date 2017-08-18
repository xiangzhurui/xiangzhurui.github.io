---
layout: drafts
title: Elasticsearch Java REST Client 概览
date: 2017-08-18 13:30:14
tags: [Elasticsearch,翻译,API 文档, REST]
---

> 本文内容来源于 Java REST Client https://www.elastic.co/guide/en/elasticsearch/client/java-rest/5.5/index.html

## 概述

`Elasticsearch Java REST Client` 是个官方的低层级的Elasticsearch客户端。 允许通过http与Elasticsearch集群进行通信。 兼容所有版本的Elasticsearch。
Java REST 客户端内部使用Apache Http Async客户端发送http请求。

## 功能

低级客户端的功能包括：
- 最小化依赖
- 跨所有可用节点的负载均衡
- 节点故障时的自动进行故障切换和响应特定代码
- 失败连接处理（一个节点是否是坏节点取决于连续失败的次数;失败的次数越多，客户端再次使用同一个节点之前的等待时间越长）
- 持久连接
- 跟踪记录请求和响应
- 可配置自动发现集群节点

## 开始使用

### Maven 仓库
低级 Java REST 客户端存在于 Maven 中央仓库里。它最低的Java版本要求是 JDK1.8。该客户端的发布周期与 Elasticsearch 的核心程序一致（存在具有相同版本号的 elasticsearch）。但该客户端事实上是与elasticsearch版本无强关联（当客户端与服务端版本可以不一致时也可使用）。

### Maven 配置
将以下内容添加到 `pom.xml` 文件里
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>rest</artifactId>
    <version>5.5.2</version>
</dependency>
```

### Gradle 配置

将以下内容添加到 `build.gradle` 文件：

```groove
dependencies {
    compile 'org.elasticsearch.client:rest:5.5.2'
}
```
### 初始化

可以通过`RestClient#builder(HttpHost...)`静态方法创建的 `RestClientBuilder` 类来构建 `RestClient` 实例。 唯一必需的参数是客户端将与之通信的一个或多个主机，作为 [HttpHost](https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/HttpHost.html) 的实例提供，如下所示：
```java
RestClient restClient = RestClient.builder(
                new HttpHost("localhost", 9200, "http"),
                new HttpHost("localhost", 9201, "http")).build();
```
`RestClient` 类是线程安全的，理想状态与使用它的应用程序具有相同的生命周期。 重要的是，当不再需要它时，它将被关闭，以便它所使用的所有资源得到正确释放，以及底层的http客户端实例及其线程：

```java
restClient.close();
```

`RestClientBuilder`还允许在构建`RestClient`实例时设置以下可选的配置参数：

- `setDefaultHeaders`
默认标头需要与每个请求一起发送，以防止每个请求都必须指定它们
- `setMaxRetryTimeoutMillis`
在为同一请求进行多次尝试时应该遵守的超时。 默认值为30秒，与默认套接字超时相同。 如果套接字超时被定制，则应该相应地调整最大重试超时
- `setFailureListener`
每当节点发生故障时收到通知的侦听器，以防需要采取措施。 启用嗅探故障时内部使用
- `setRequestConfigCallback`
回调允许修改默认请求配置（例如，请求超时，身份验证或任何[org.apache.http.client.config.RequestConfig.Builder](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/client/config/RequestConfig.Builder.html)允许设置的内容）
- `setHttpClientConfigCallback`
回调允许修改http客户端配置（例如通过ssl进行的加密通信，或任何[org.apache.http.impl.nio.client.HttpAsyncClientBuilder](http://hc.apache.org/httpcomponents-asyncclient-dev/httpasyncclient/apidocs/org/apache/http/impl/nio/client/HttpAsyncClientBuilder.html)允许设置的任何内容）

### 执行请求
一旦一个 `RestClient` 已经北创建，请求就可以通过一个名为 `performRequest` 和 `performRequestAsync` 的方法创建。`performRequest` 方法是同步的，他直接返回 `Response`, 这意味着客户端将会阻塞等待响应的返回。`performRequestAsync` 方法返回 `void` 然后接受一个额外的 `ResponseListener` 作为参数，然后方法价格会异步执行，ResponseListener将会通知完成或失败的执行结果。
```java
// Synchronous variants
Response performRequest(String method, String endpoint,
                        Header... headers)
    throws IOException;

Response performRequest(String method, String endpoint,
                        Map<String, String> params, Header... headers)
    throws IOException;

Response performRequest(String method, String endpoint,
                        Map<String, String> params,
                        HttpEntity entity,
                        Header... headers)
    throws IOException;

Response performRequest(String method, String endpoint,
                        Map<String, String> params,
                        HttpEntity entity,
                        HttpAsyncResponseConsumerFactory responseConsumerFactory,
                        Header... headers)
    throws IOException;

// Asynchronous variants
void performRequestAsync(String method, String endpoint,
                         ResponseListener responseListener,
                         Header... headers);

void performRequestAsync(String method, String endpoint,
                         Map<String, String> params,
                         ResponseListener responseListener,
                         Header... headers);

void performRequestAsync(String method, String endpoint,
                         Map<String, String> params,
                         HttpEntity entity,
                         ResponseListener responseListener,
                         Header... headers);

void performRequestAsync(String method, String endpoint,
                         Map<String, String> params,
                         HttpEntity entity,
                         HttpAsyncResponseConsumerFactory responseConsumerFactory,
                         ResponseListener responseListener,
                         Header... headers);
```

### 请求参数
以下是不同方法接受的参数：

- method
http方法或动作
endpoint
请求路径，Elasticsearch API 定义的路径（例如/_cluster/health ）
params
the optional parameters to be sent as querystring parameters
entity
the optional request body enclosed in an org.apache.http.HttpEntity object
responseConsumerFactory
the optional factory that is used to create an org.apache.http.nio.protocol.HttpAsyncResponseConsumer callback instance per request attempt. Controls how the response body gets streamed from a non-blocking HTTP connection on the client side. When not provided, the default implementation is used which buffers the whole response body in heap memory, up to 100 MB
responseListener
the listener to be notified upon asynchronous request success or failure
headers
optional request headers
