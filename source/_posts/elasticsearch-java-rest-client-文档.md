---
layout: drafts
title: 'Elasticsearch Java REST Client 文档[翻译]'
tags:
  - Elasticsearch
  - 翻译
  - API 文档
  - REST
date: 2017-08-18 13:30:14
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
- endpoint
请求路径，Elasticsearch API 定义的路径（例如`/_cluster/health` ）
- params
可选参数作为querystring参数发送
- entity
包含在org.apache.http.HttpEntity对象中的可选请求主体
- responseConsumerFactory
用于org.apache.http.nio.protocol.HttpAsyncResponseConsumer 每次请求尝试创建回调实例的可选工厂 。控制响应主体如何从客户端上的非阻塞HTTP连接流式传输。当没有提供时，使用默认实现来缓冲堆内存中的整个响应体，最多100 MB
- responseListener
侦听器在异步请求成功或失败时被通知
- headers
可选请求标头

### 读取响应

Response对象，要么由同步的performRequest的方法返回，要么作为 ResponseListener#onSuccess(Response)的参数，通过这个被包装的HTTP客户端返回的响应对象和公开以下信息：

- getRequestLine
有关执行的请求的信息
- getHost
返回响应的主机
- getStatusLine
响应状态行
- getHeaders
响应标题，也可以通过名称检索 getHeader(String)
- getEntity
响应体包围在一个 org.apache.http.HttpEntity 对象中
执行请求时，会抛出异常（或ResponseListener#onFailure(Exception)在以下情况下作为参数接收：

- IOException
通信问题（例如SocketTimeoutException等）
- ResponseException
已返回响应，但其状态代码指示错误（不是2xx）。A ResponseException源自有效的http响应，因此它暴露了其对应的Response对象，该对象允许访问返回的响应。

**NOTE**: 一个 ResponseException 是`不`抛出`HEAD`返回一个请求的404状态代码，因为它是一个预期的HEAD，仅仅表示该资源未找到响应。所有其他的HTTP方法（例如GET）抛出ResponseException了404回应，除非该ignore参数包含404。ignore是一个特殊的客户端参数，不会发送到Elasticsearch，并包含一个逗号分隔的错误状态代码列表。它允许控制是否将某个错误状态代码视为预期响应而不是异常。这对于get API可以返回是有用的404 当文档丢失时，在这种情况下，响应主体不会包含错误，而是通常的get api响应，只是没有找到文档。

> 一下内容待续：https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/_example_requests.html
