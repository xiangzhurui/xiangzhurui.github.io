---
layout: drafts
title: 'Elasticsearch Java REST Client 文档[译文]'
tags:
  - Elasticsearch
  - 翻译
  - API 文档
  - REST
date: 2017-08-18 13:30:14
update: 2017-09-05 12:40:00
categories: [翻译,Elasticsearch]
---


> 本文文档适用于 Elasticsearch 5.5。
> 原文链接：[Java REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/5.5/index.html)
> 译文链接：[Elasticsearch Java REST Client 文档](./)

## 概述

`Elasticsearch Java REST Client` 是个官方的低层级的 Elasticsearch 客户端。允许通过 http 与Elasticsearch集群进行通信。 并且兼容所有版本的 Elasticsearch。
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

## 入门

### Maven 仓库
低级 Java REST 客户端存在于 Maven 中央仓库里。它最低的Java版本要求是 JDK1.8。该客户端的发布周期与 Elasticsearch 的核心程序一致（存在具有相同版本号的 Elasticsearch ）。但该客户端事实上是与 Elasticsearch 版本无强关联（当客户端与服务端版本可以不一致时也可使用）。

#### Maven 配置
将以下内容添加到 `pom.xml` 文件里
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>rest</artifactId>
    <version>5.5.2</version>
</dependency>
```

#### Gradle 配置

使用 `gradle` 作依赖管理时,你可以向这样配置 `sniffer` 依赖，将下列内容添加到你的 `build.gradle` 文件里：

```groove
dependencies {
    compile 'org.elasticsearch.client:rest:5.5.2'
}
```

### 依赖项

低级 Java REST 客户端 内部使用 Apache Http Async Client 发送 http 请求. 他依赖以下部件,
亦即异步 http 客户端和它自己的传递依赖项

* org.apache.httpcomponents:httpasyncclient
* org.apache.httpcomponents:httpcore-nio
* org.apache.httpcomponents:httpclient
* org.apache.httpcomponents:httpcore
* commons-codec:commons-codec
* commons-logging:commons-logging

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

一旦一个 `RestClient` 实例被创建。就可以通过调用一个可用 `performRequest` 和 `performRequestAsync` 方法预期重载方法来发送请求。`performRequest` 方法是同步的，他直接返回 `Response`, 这意味着客户端将会阻塞等待响应的返回。`performRequestAsync` 方法返回 `void` 并接受一个额外的 `ResponseListener` 作为参数，他是异步执行的方法。在方法执行完成或失败时将会通知传入的监听器（译注：ResponseListener）。

```java
// 同步请求方法
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

// 异步请求方法
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

**请求参数**

以下是对不同方法的参数说明：

- `method`
  http方法或动作
- `endpoint`
  请求路径，Elasticsearch API 定义的路径（例如`/_cluster/health` ）
- `params`
  可选参数。作为querystring参数发送。
- `entity`
  可选参数。封装在`org.apache.http.HttpEntity`对象中的请求主体
- `responseConsumerFactory`
  可选参数。用于每次请求接受的创建 [org.apache.http.nio.protocol.HttpAsyncResponseConsumer](http://hc.apache.org/httpcomponents-core-ga/httpcore-nio/apidocs/org/apache/http/nio/protocol/HttpAsyncResponseConsumer.html) 回调实例的工厂类 。在客户端控制如何从一个非阻塞HTTP连接上获取响应主体的数据流。当没有使用该参数时，默认实现将整个响应体缓冲到堆内存中，上限为 100 MB
- `responseListener`
  异步请求成功或失败时收到通知的监听器。
- `headers`
  可选参数。http 请求头。

### 读取响应

`Response` 对象，要么由同步的 `performRequest` 的方法返回，要么作为 `ResponseListener#onSuccess(Response)` 的参数持有，响应对象包装了由HTTP客户端返回并提供的公开信息：

- `getRequestLine`
  有关执行的请求的信息
- `getHost`
  返回响应的主机
- `getStatusLine`
  响应状态行
- `getHeaders`
  响应标题，也可以使用 `getHeader(String)`方法按名称获取
- `getEntity`
  响应体包含在 `org.apache.http.HttpEntity` 对象中。

在执行请求时，以下情况下将抛出异常（或者作为参数传入到  `ResponseListener#onFailure(Exception)` 里）：

- `IOException`
  通信问题（例如：SocketTimeoutException 等）
- `ResponseException`
  有响应返回，但其状态代码表明错误（不是`2xx`）。一个 `ResponseException` 来自一个有效的 http 响应，因此它也暴露了其对应的一个允许被访问的 `Response` 对象。

**注意**: 对于返回 `404` 状态码的 `HEAD` 请求 **不会** 抛出 `ResponseException` 异常，因为这是一个预期内的结果，这一响应仅表示该资源未被找到。M所有其他的 HTTP 方法，当响应 `404`时，（例如：`GET`）将抛出 `ResponseException` 异常，除非 `ignore` 参数包含 `404`。`ignore` 是一个特殊的客户端参数，它包含逗号分隔的错误状态码列表，并且不会被发送到 Elasticsearch。这一参数允许控制是否将某些错误状态代码视为预期响应而不是异常。这是有用的，例如，当只是因为文档没有被找到时 get api 可以返回 404，响应主体不会包含错误，而是返回正常的 get 响应。


### 请求样例

这里有一对样例：

```java
Response response = restClient.performRequest("GET", "/",
        Collections.singletonMap("pretty", "true"));
System.out.println(EntityUtils.toString(response.getEntity()));

//index a document
HttpEntity entity = new NStringEntity(
        "{\n" +
        "    \"user\" : \"kimchy\",\n" +
        "    \"post_date\" : \"2009-11-15T14:12:12\",\n" +
        "    \"message\" : \"trying out Elasticsearch\"\n" +
        "}", ContentType.APPLICATION_JSON);

Response indexResponse = restClient.performRequest(
        "PUT",
        "/twitter/tweet/1",
        Collections.<String, String>emptyMap(),
        entity);
```

> 为 HttpEntity 指定的 ContentType 很重要，因为它将用于设置 Content-Type 头，然后 Elasticsearch 才可以方便的正确解析内容。未来版本的 Elasticsearch 将强制要求将其设置的正确性。

注意此低级客户端并没有制定额外的帮助编码和解码 json 的工具。用户可以自由的使用他们自己喜欢的类库。

底层 Apache Async Http Client 提供允许不同请求题格式（stream, byte array, string 等）的 [org.apache.http.HttpEntity](https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/HttpEntity.html) 实现。对于读取响应体，`HttpEntity＃getContent`方法很方便，它从先前的响应体缓冲里读取并返回一个 `InputStream` 对象。作为替代方案，可以提供一个自定义 [org.apache.http.nio.protocol.HttpAsyncResponseConsumer](http://hc.apache.org/httpcomponents-core-ga/httpcore-nio/apidocs/org/apache/http/nio/protocol/HttpAsyncResponseConsumer.html  )来控制如何读取和缓冲字节。

一下是如何发送异步请求的一个简单示例：

```java
int numRequests = 10;
final CountDownLatch latch = new CountDownLatch(numRequests);

for (int i = 0; i < numRequests; i++) {
    restClient.performRequestAsync(
        "PUT",
        "/twitter/tweet/" + i,
        Collections.<String, String>emptyMap(),
        //假设文档被存储在实体数组中
        entities[i],
        new ResponseListener() {
            @Override
            public void onSuccess(Response response) {
                System.out.println(response);
                latch.countDown();
            }

            @Override
            public void onFailure(Exception exception) {
                latch.countDown();
            }
        }
    );
}

//等待所有请求完成。
latch.await();
```

### 日志

Java REST 客户端使用与 Apache Async Http Client 相同的日志记录库：[Apache Commons Logging](https://commons.apache.org/proper/commons-logging/)，它支持许多流行的日志记录实现。启用日志记录的java包是客户机本身的 `org.elasticsearch.client` 和嗅探器的 `org.elasticsearch.client.sniffer`。

还可以在 crul 范式里启用请求跟踪记录来记录每个请求和相应的响应。这在调试时很方便，例如在需要手动执行请求以检查它是否仍然产生与之相同的响应的情况下。 启用跟踪程序包的跟踪记录以打印出这样的日志行。 需要注意的是，这种类型的日志记录的性能开销是昂贵的，他不应该在生产环境中始终启用，而是仅在需要时暂时使用。

## 通用配置

`RestClientBuilder` 提供了一个 `RequestConfigCallback` 和一个 `HttpClientConfigCallback`来对 `Apache Async Http Client` 对外的公共方法进行任何定制。这些回调对象使得在不用覆盖 “RestClient” 初始化的其他所有默认配置的情况下修改一些客户端的特殊行为成为可能。本节介绍一些需要对低级`Java REST Client`的额外配置的常见场景。

### 超时
请求超时参数的配置可以通过在 `RestClient` 的建造者(译注：实际为即 *RestClientBuilder* )在建造实例的过程张提供一个 `RequestConfigCallback 实例完成。该接口有一个接收 [org.apache.http.client.config.RequestConfig.Builder](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/client/config/RequestConfig.Builder.html) 实例作为参数并具有相同返回值类型的方法。请求建造者(译注: *RestClientBuilder* )可以被修改并返回修改后的对象。在下面的样例中，我们增加链接超时时间（默认值为1秒）和套接字超时时间（默认值为30秒）。同样的我们也相应的调整醉倒重试超时时间（默认值也是30秒）。

```java
RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200))
        .setRequestConfigCallback(new RestClientBuilder.RequestConfigCallback() {
            @Override
            public RequestConfig.Builder customizeRequestConfig(RequestConfig.Builder requestConfigBuilder) {
                return requestConfigBuilder.setConnectTimeout(5000)
                        .setSocketTimeout(60000);
            }
        })
        .setMaxRetryTimeoutMillis(60000)
        .build();
```

### 线程数
`Apache Http Async Client` 默认情况下启动一个调度线程，和一定数量的由连接管理器使用的工作线程，工作线程数量与本机检测到的处理器的数量值一致(该值取决于方法 `Runtime.getRuntime().availableProcessors()` 的返回结果)。线程数量的修改可以以以下方式进行:
```java
RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200))
        .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                return httpClientBuilder.setDefaultIOReactorConfig(
                        IOReactorConfig.custom().setIoThreadCount(1).build());
            }
        })
        .build();
```

### 基本身份验证

配置基本认证可以通过在 `RestClient` 的建造者建造时提供一个 `HttpClientConfigCallback` 来实现。这个接口有一个接收
 [org.apache.http.impl.nio.client.HttpAsyncClientBuilder](https://hc.apache.org/httpcomponents-asyncclient-dev/httpasyncclient/apidocs/org/apache/http/impl/nio/client/HttpAsyncClientBuilder.html) 实例作为参数并具有相同返回值类型的方法。 这个 http 客户端建造者可以被修改并返回修改后的建造者。 在下面的样例中我们设置一个需要基本身份验证的默认凭据。

```java
final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
credentialsProvider.setCredentials(AuthScope.ANY,
        new UsernamePasswordCredentials("user", "password"));

RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200))
        .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
            }
        })
        .build();
```

您可以禁用 Preemptive Authentication，这意味着每个请求都将发送没有授权头来查看是否被接受，当收到HTTP 401响应后，它将使用基本身份验证头重新发送完全相同的请求。如果你想这样做，那么你可以通过 `HttpAsyncClientBuilder` 禁用它：

```java
final CredentialsProvider credentialsProvider = new BasicCredentialsProvider();
credentialsProvider.setCredentials(AuthScope.ANY,
        new UsernamePasswordCredentials("user", "password"));

RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200))
        .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                // disable preemptive authentication
                httpClientBuilder.disableAuthCaching();
                return httpClientBuilder.setDefaultCredentialsProvider(credentialsProvider);
            }
        })
        .build();
```

### 加密通信

加密通讯可以也可以通过一个 `HttpClientConfigCallback` 进行配置。接口 [org.apache.http.impl.nio.client.HttpAsyncClientBuilder](https://hc.apache.org/httpcomponents-asyncclient-dev/httpasyncclient/apidocs/org/apache/http/impl/nio/client/HttpAsyncClientBuilder.html) 接受一个参数的公开的多个方法来配置加密通信：`setSSLContext`, `setSSLSessionStrategy` 和 `setConnectionManager`, 以上方法按照最不重要的顺序排序. 下面是一个样例：

```java
KeyStore keystore = KeyStore.getInstance("jks");
try (InputStream is = Files.newInputStream(keyStorePath)) {
    keystore.load(is, keyStorePass.toCharArray());
}

RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200))
        .setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
            @Override
            public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
                return httpClientBuilder.setSSLContext(sslcontext);
            }
        })
        .build();

```

### 其他

对于其他任何需要配置项, 可以查阅 `Apache HttpAsyncClient` 文档： [https://hc.apache.org/httpcomponents-asyncclient-4.1.x/](https://hc.apache.org/httpcomponents-asyncclient-4.1.x/)

## 嗅探器（*Sniffer*）

这是一个小型的包，他允许从一个正在运行中的 Elasticsearch 集群里自动发现节点，并将其设置更新一个现有的RestClient实例上。默认情况下他使用使用`Nodes Info` api检索属于集群的节点，并使用`jackson`解析获取的`json`响应。

该库与Elasticsearch 2.x及以上兼容。

### Maven 仓库
低级REST客户端与 Elasticsearch 的发行周期相同。 可以使用你希望使用的嗅探器（Sniffer）版本替换版本，这里使用 `5.5.2` 版本。嗅探器版本和与 Elasticsearch 通信的 REST 客户端的版本之间没有关系。嗅探器支持从 Elasticsearch 2.x及以上获取节点列表。

### Maven 配置

使用 `Maven` 作依赖管理时,你可以向这样配置 `sniffer` 依赖，将下列内容添加到你的 `pom.xml` 文件里：

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>sniffer</artifactId>
    <version>5.5.2</version>
</dependency>
```
### Gradle 配置
使用 `gradle` 作依赖管理时,你可以向这样配置 `sniffer` 依赖，将下列内容添加到你的 `build.gradle` 文件里：

```
dependencies {
    compile 'org.elasticsearch.client:sniffer:5.5.2'
}
```

### 使用
只要一个 `RestClient` 实例被创建了，某个 `Sniffer` 对象就可以关联到它上面。`Sniffer` 将使用他所关联的 `RestClient` 实例去定期（默认情况下位每5分钟）从集群中获取节点列表，然后调用 `RestClient#setHosts` 将节点列表更新这个 `RestClient` 实例上去。
```java
Sniffer sniffer = Sniffer.builder(restClient).build();
```
关闭 `Sniffer` 是必要的，这可以使其后台线程正确的关闭，并且释放他持有的所有资源。该 `Sniffer` 对象应该与 `RestClient` 独享具有相同的生命周期，并在 `RestClient` 客户端关闭之前关闭：
```java
sniffer.close();
restClient.close();
```

`Elasticsearch Nodes Info api` 不会返回连接节点时使用的协议类型，除非 `host:port` 里被体现了加密协议(*译注：即是https*)，所以默认情况下使用 `http` 。如果需要使用 `https` 替换他，那么 `ElasticsearchHostsSniffer` 对象需要按照以下方式手动创建：

```java
HostsSniffer hostsSniffer = new ElasticsearchHostsSniffer(restClient,
        ElasticsearchHostsSniffer.DEFAULT_SNIFF_REQUEST_TIMEOUT,
        ElasticsearchHostsSniffer.Scheme.HTTPS);
Sniffer sniffer = Sniffer.builder(restClient)
        .setHostsSniffer(hostsSniffer).build();
```
同样的我们也可以自定义 `sniffRequestTimeout` 参数，该参数默认值为1秒。当调用 `Nodes Info` api 时，这个 `timeout` 参数被用作querystring参数使用，因此当服务端超时时，仍然会有一个有效响应返回，尽管他可能只包含集群的一部分，所有节点的一个子集，余下的那些将在随后响应。另外，一个自定义的 `HostsSniffer` 实现可以提供一些高级用法，比如从外部来源获取主机。

默认情况下 `Sniffer` 每五分钟更新一次。这个间隔时间（单位为毫秒）可以以如下方式设置自定义值：

```java
Sniffer sniffer = Sniffer.builder(restClient)
        .setSniffIntervalMillis(60000).build();
```
也可以启用故障嗅探，这意味着在每次故障之后节点列表将被直接更新，而不是在普通的嗅探轮询里更新。在这种情况下，需要首先创建一个 `SniffOnFailureListener` 并且提供给 `RestClient` 实例化时使用。当然一旦 `Sniffer` 在这之后被创建时，他需要关联到上文的同一个 `SniffOnFailureListener` 实例上，这个每一个故障都将通知到这个 `SniffOnFailureListener`，并且使用 `Sniffer` 执行上文上所述的嗅探更新操作。

```java
SniffOnFailureListener sniffOnFailureListener = new SniffOnFailureListener();
RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200))
        .setFailureListener(sniffOnFailureListener).build();
Sniffer sniffer = Sniffer.builder(restClient).build();
sniffOnFailureListener.setSniffer(sniffer);
```
当使用故障嗅探器时，在故障发生后不只是会更新节点，还会执行额外添加的比正常掌控下更早嗅探计划，默认情况下嗅探更新在故障发生1分钟后执行，假如节点将会恢复正常，那么我们希望尽快获知到。如下所述，轮训间隔可以在 `Sniffer` 创建时自定义设置：

```java
Sniffer sniffer = Sniffer.builder(restClient)
        .setSniffAfterFailureDelayMillis(30000).build();
```
**注意** 如果没有启用失败监听器(*`setFailureListener`*)，则此最后一个配置参数(*`SniffAfterFailureDelayMillis`*)将不会生效


### 许可

Copyright 2013-2016 Elasticsearch

根据 `Apache License, Version 2.0`（“License”）许可; 您不得使用此文件，除非符合许可证。您可以获得许可证的副本

```
http://www.apache.org/licenses/LICENSE-2.0
```

除非由书面的法律要求或许可，根据许可证分发的软件以**“按现状”基础分发，不附带任何明示或暗示的担保或条件**。有关许可证下的权限和限制的特定语言，请参阅许可证。
