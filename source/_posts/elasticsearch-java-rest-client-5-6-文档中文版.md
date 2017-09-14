---
title: Elasticsearch Java REST Client 5.6 文档中文版
categories:
  - 翻译
  - Elasticsearch
tags:
  - Elasticsearch
  - Elasticsearch 5.6
  - 翻译
  - API 文档
  - REST
date: 2017-09-13 11:39:49
---


## 概览

Java REST客户端有两种风格:
* [Java 低级 REST 客户端](#Java-低级-REST-客户端)：Elasticsearch的官方低级客户端。 它允许通过http与Elasticsearch集群进行通信。请求编码和响应解码保留给用户实现。它与所有 Elasticsearch 版本兼容。
* [Java 高级 REST 客户端](#Java-高级-REST-客户端)：Elasticsearch的官方高级客户端。 基于低级客户端，它提供特定的方法的API，并处理请求编码和响应解码。

## Java 低级 REST 客户端

低级客户端的功能：
* 最小依赖
* 所有可用节点间的负载均衡
* 节点故障和特定响应代码时的故障切换
* 故障连接策略 (是否重新连接故障节点取决于连续失败多少次；失败次数越多，在再次尝试同一个节点之前，客户端等待的时间越长）
* 持久化连接
* 跟踪记录请求和响应
* 自动[发现集群节点](#嗅探器)选项

### 起步
本节介绍了如何在应用程序中使用开始使用低级REST客户端。

#### Javadoc
低级 REST 客户端的 javadoc 可以在这里找到  [https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-client/5.6.0/index.html](https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-client/5.6.0/index.html) 。

#### Maven 仓库
低级 Java REST 客户端被托管在 Maven 中央仓库里。所需的最低Java版本为 `1.7`。

低级 REST 客户端与 elasticsearch 的发行周期相同。可以使用期望的客户端版本替换，但必须是 `5.0.0-alpha4` 之后的版本。客户端版本与其通信的 Elasticsearch 版本之间没有关联。低级 REST 客户端兼容所有 Elasticsearch 版本。

##### Maven 配置
若使用 Maven 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 `pom.xml` 文件里：
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client</artifactId>
    <version>5.6.0</version>
</dependency>
```

##### Gradle 配置
若使用 gradle 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 `build.gradle` 文件里：

```groovy
dependencies {
    compile 'org.elasticsearch.client:elasticsearch-rest-client:5.6.0'
}
```

#### 依赖项
低级 Java REST 客户端内部使用 [Apache Http Async Client](http://hc.apache.org/httpcomponents-asyncclient-dev/) 发送 http 请求。它依赖以下工件，即异步 http 客户端与其自身的传递依赖项：

* org.apache.httpcomponents:httpasyncclient
* org.apache.httpcomponents:httpcore-nio
* org.apache.httpcomponents:httpclient
* org.apache.httpcomponents:httpcore
* commons-codec:commons-codec
* commons-logging:commons-logging

#### Shading
为了避免版本冲突，可以在单个JAR文件（有时称为“uber JAR”或“fat JAR”）中将客户端中的依赖项隐藏并打包在客户端中。隐藏依赖关系包括在将其内容（资源文件和Java类文件）与其低级Java REST客户端放在同一个JAR文件中之前重新命名其一些软件包。 Shading JAR 可以通过Gradle和Maven的第三方插件完成。

请注意，遮蔽JAR也有影响。 例如，遮蔽 Commons Logging层意味着第三方日志记录后端也需要被遮蔽。

##### Maven 配置
你可以这样配置 Maven [Shade](https://maven.apache.org/plugins/maven-shade-plugin/index.html) 插件 。将下列内容添加到你的 `pom.xml` 文件里：

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-shade-plugin</artifactId>
            <version>3.1.0</version>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals><goal>shade</goal></goals>
                    <configuration>
                        <relocations>
                            <relocation>
                                <pattern>org.apache.http</pattern>
                                <shadedPattern>hidden.org.apache.http</shadedPattern>
                            </relocation>
                            <relocation>
                                <pattern>org.apache.logging</pattern>
                                <shadedPattern>hidden.org.apache.logging</shadedPattern>
                            </relocation>
                            <relocation>
                                <pattern>org.apache.commons.codec</pattern>
                                <shadedPattern>hidden.org.apache.commons.codec</shadedPattern>
                            </relocation>
                            <relocation>
                                <pattern>org.apache.commons.logging</pattern>
                                <shadedPattern>hidden.org.apache.commons.logging</shadedPattern>
                            </relocation>
                        </relocations>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```
##### Gradle 配置
你可以这样配置 Gradle [ShadowJar](https://github.com/johnrengelman/shadow) 插件. 添加以下内容到你的 `build.gradle` 文件中：

```groovy
shadowJar {
    relocate 'org.apache.http', 'hidden.org.apache.http'
    relocate 'org.apache.logging', 'hidden.org.apache.logging'
    relocate 'org.apache.commons.codec', 'hidden.org.apache.commons.codec'
    relocate 'org.apache.commons.logging', 'hidden.org.apache.commons.logging'
}
```

#### 初始化
`RestClient` 实例可以通过相应的 `RestClientBuilder` 类来构建,通过静态方法 `RestClient#builder(HttpHost...)` 创建。唯一必需的参数是客户端将与之通信的一个或多个主机，以 [HttpHost](https://hc.apache.org/httpcomponents-core-ga/httpcore/apidocs/org/apache/http/HttpHost.html) 实例的方式提供给建造者，如下所示：
```java
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"),
        new HttpHost("localhost", 9201, "http")).build();
```
`RestClient` 类是线程安全的，理想状态下它与使用它的应用程序具有相同的生命周期。当不再使用时关闭它是非常必要的，如此才能是所有被使用到的资源正确的被释放，当然在底层太实用 http 客户端及其线程执行操作：
```java
restClient.close();
```

`RestClientBuilder` 还允许在构建 `RestClient` 实例时设置以下的可选配置参数：

```java
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
Header[] defaultHeaders = new Header[]{
        new BasicHeader("header", "value")
    };
builder.setDefaultHeaders(defaultHeaders); // 设置每个请求发送时的默认头文件，以免每个请求都必须指定它们。
```

设置

```java
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setMaxRetryTimeoutMillis(10000);
```
> 设置在同一请求进行多次尝试时应该遵守的超时时间。 默认值为30秒，与默认套接字超时相同。 如果自定义设置了套接字超时，则应该相应地调整最大重试超时。

```java
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setFailureListener(new RestClient.FailureListener() {
    @Override
    public void onFailure(HttpHost host) {
        //设置每次节点发生故障时收到通知的侦听器，触发如果需要采取的行动。内部嗅探到故障时被启用。
    }
});
```

```java
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setRequestConfigCallback(new RestClientBuilder.RequestConfigCallback() {
        @Override
        public RequestConfig.Builder customizeRequestConfig(RequestConfig.Builder requestConfigBuilder) {
            return requestConfigBuilder.setSocketTimeout(10000);
        }
    });//
```

> 设置允许修改默认请求配置的回调（例如：请求超时，认证，或者其他 [org.apache.http.client.config.RequestConfig.Builder](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/client/config/RequestConfig.Builder.html) 允许的设置）。

```java
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
    @Override
    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
        return httpClientBuilder.setProxy(new HttpHost("proxy", 9000, "http")); //
    }
});
```

> 设置允许修改 http 客户端配置的回调（例如：ssl 加密通讯，或其他任何 [org.apache.http.impl.nio.client.HttpAsyncClientBuilder](http://hc.apache.org/httpcomponents-asyncclient-dev/httpasyncclient/apidocs/org/apache/http/impl/nio/client/HttpAsyncClientBuilder.html) 允许的设置）

#### 执行请求
#### 读取响应
#### 日志记录

### 通用配置
#### 超时
#### 线程数
#### 基本认证
#### 加密传输
#### 其他

### 嗅探器
这是个小型库，可以允许从一个正在运行的 Elasticsearch 集群上自动发现节点并将节点列表更新到已经存在的 `RestClient` 实例上。
它默认使用 Nodes Info api 检索属于集群的节点，并使用jackson解析获取的json响应。

与Elasticsearch 2.x及以上兼容。

#### Javadoc

REST 客户端嗅探器的 javadoc 可以在这里找到 [https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-client-sniffer/5.6.0/index.html](https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-client-sniffer/5.6.0/index.html)

#### Maven 仓库

REST 客户端嗅探器与 elasticsearch 的发行周期相同。可以使用期望的嗅探器版本替换，但必须是 `5.0.0-alpha4` 之后的版本。嗅探器版本与其通信的 Elasticsearch 版本之间没有关联。嗅探器支持从 `elasticsearch 2.x` 及以上的版本上获取节点列表。

##### Maven 配置
若使用 Maven 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 `pom.xml` 文件里：

```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client-sniffer</artifactId>
    <version>5.6.0</version>
</dependency>
```

##### Gradle 配置
若使用 gradle 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 `build.gradle` 文件里：
```groovy
dependencies {
    compile 'org.elasticsearch.client:elasticsearch-rest-client-sniffer:5.6.0'
}
```
#### 用法

一旦创建了RestClient实例，如“初始化”中所示，可以将嗅探器与之相关联。 `Sniffer` 将使用关联的 `RestClient` 定期（默认为每5分钟）从集群中获取当前可用的节点列表，并通过调用 `RestClient＃setHosts` 来更新它们。

```java
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
Sniffer sniffer = Sniffer.builder(restClient).build();
```

It is important to close the Sniffer so that its background thread gets properly shutdown and all of its resources are released. The Sniffer object should have the same lifecycle as the RestClient and get closed right before the client:

```java
sniffer.close();
restClient.close();
```

The Sniffer updates the nodes by default every 5 minutes. This interval can be customized by providing it (in milliseconds) as follows:

```java
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
Sniffer sniffer = Sniffer.builder(restClient)
        .setSniffIntervalMillis(60000).build();
```

It is also possible to enable sniffing on failure, meaning that after each failure the nodes list gets updated straightaway rather than at the following ordinary sniffing round. In this case a SniffOnFailureListener needs to be created at first and provided at RestClient creation. Also once the Sniffer is later created, it needs to be associated with that same SniffOnFailureListener instance, which will be notified at each failure and use the Sniffer to perform the additional sniffing round as described.

```java
SniffOnFailureListener sniffOnFailureListener = new SniffOnFailureListener();
RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200))
        .setFailureListener(sniffOnFailureListener)
        .build();
Sniffer sniffer = Sniffer.builder(restClient)
        .setSniffAfterFailureDelayMillis(30000)
        .build();
sniffOnFailureListener.setSniffer(sniffer);
```

Set the failure listener to the RestClient instance



When sniffing on failure, not only do the nodes get updated after each failure, but an additional sniffing round is also scheduled sooner than usual, by default one minute after the failure, assuming that things will go back to normal and we want to detect that as soon as possible. Said interval can be customized at Sniffer creation time through the setSniffAfterFailureDelayMillis method. Note that this last configuration parameter has no effect in case sniffing on failure is not enabled like explained above.



Set the Sniffer instance to the failure listener

The Elasticsearch Nodes Info api doesn’t return the protocol to use when connecting to the nodes but only their host:port key-pair, hence http is used by default. In case https should be used instead, the ElasticsearchHostsSniffer instance has to be manually created and provided as follows:

```java
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
HostsSniffer hostsSniffer = new ElasticsearchHostsSniffer(
        restClient,
        ElasticsearchHostsSniffer.DEFAULT_SNIFF_REQUEST_TIMEOUT,
        ElasticsearchHostsSniffer.Scheme.HTTPS);
Sniffer sniffer = Sniffer.builder(restClient)
        .setHostsSniffer(hostsSniffer).build();
```

In the same way it is also possible to customize the sniffRequestTimeout, which defaults to one second. That is the timeout parameter provided as a querystring parameter when calling the Nodes Info api, so that when the timeout expires on the server side, a valid response is still returned although it may contain only a subset of the nodes that are part of the cluster, the ones that have responded until then.

```java
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
HostsSniffer hostsSniffer = new ElasticsearchHostsSniffer(
        restClient,
        TimeUnit.SECONDS.toMillis(5),
        ElasticsearchHostsSniffer.Scheme.HTTP);
Sniffer sniffer = Sniffer.builder(restClient)
        .setHostsSniffer(hostsSniffer).build();
```
Also, a custom HostsSniffer implementation can be provided for advanced use-cases that may require fetching the hosts from external sources rather than from Elasticsearch:
```java
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
HostsSniffer hostsSniffer = new HostsSniffer() {
    @Override
    public List<HttpHost> sniffHosts() throws IOException {
        return null; //1
    }
};
Sniffer sniffer = Sniffer.builder(restClient)
        .setHostsSniffer(hostsSniffer).build();
```

> 1. 从外部源获取主机

### 许可证

Copyright 2013-2016 Elasticsearch

根据 `Apache License, Version 2.0`（“License”）许可; 您不得使用此文件，除非符合许可证。您可以获得许可证的副本

```
http://www.apache.org/licenses/LICENSE-2.0
```

除非由书面的法律要求或许可，根据许可证分发的软件以**“按现状”基础分发，不附带任何明示或暗示的担保或条件**。有关许可证下的权限和限制的特定语言，请参阅许可证。


## Java 高级 REST 客户端
Java高级REST客户端可以在Java Low Level REST客户机之上工作。 其主要目标是公开特定方法的API，接受请求对象作为参数并返回响应对象，以便客户端自己处理请求编组和响应解组。

每个API可以同步或异步地调用。 同步方法返回一个响应对象，而名称以 `async` 后缀结尾的异步方法需要收到响应或错误后才会通知（在低级别客户端管理的线程池上）的侦听器参数。

Java高级REST客户端依赖于 Elasticsearch 核心项目。 它接受与 `TransportClient` 相同的请求参数，并返回相同的响应对象。

### 起步
本节介绍了如何通过工件在应用程序中使用高级REST客户端。

#### 兼容性
Java高级REST客户端需要Java 1.8，并依赖于Elasticsearch核心项目。 客户端版本要与客户端开发的Elasticsearch版本相同。 它接受与 `TransportClient` 相同的请求参数，并返回相同的响应对象。 如果需要将应用程序从TransportClient迁移到新的REST客户端，请参阅“[迁移指南](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-level-migration.html)”。

高级客户端保证能够与运行在相同主版本和大于或等于次要版本的任何Elasticsearch节点进行通信。它不需要与它进行通信的弹性搜索节点相同的次要版本，因为它是向前兼容的，意味着它支持与之前开发的弹性搜索的更新版本进行通信。

5.6客户端可以与任何5.6.x弹性搜索节点进行通信。以前的5.x小版本，如5.5.x，5.4.x等（完全）不支持。

6.0客户端能够与任何6.x Elasticsearch节点进行通信，而6.1客户端确实能够与6.1,6.2和以后的6.x版本进行通信，但与以前的Elasticsearch节点版本通信时可能会出现不兼容问题例如6.1到6.0之间，例如6.1客户端支持而6.0节点不知道的某些API的新请求主体字段。

建议在将Elasticsearch集群升级到新的主要版本时升级高级客户端，因为REST API突破性更改可能会导致意外的结果，具体取决于请求所击中的节点，新添加的API只能由较新版本的客户端。一旦群集中的所有节点都升级到新的主版本，则客户端应当更新。

#### Javadoc
REST 高级客户端的 javadoc 可以在这里找到 [https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-high-level-client/5.6.0/index.html]( https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-high-level-client/5.6.0/index.html)

#### Maven 仓库

高级 Java REST 客户端被托管在 Maven 中央仓库里。所需的最低Java版本为 `1.8`。

高级 REST 客户端与 elasticsearch 的发行周期相同。可以使用期望的版本进行替换。

##### Maven 配置
若使用 Maven 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 `pom.xml` 文件里：
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-high-level-client</artifactId>
    <version>5.6.0</version>
</dependency>
```
##### Gradle 配置
若使用 gradle 作依赖管理，你可以这样配置依赖。将下列内容添加到你的 `build.gradle` 文件里：

```groovy
dependencies {
    compile 'org.elasticsearch.client:elasticsearch-rest-high-level-client:5.6.0'
}
```
#### 依赖项
高级 Java REST Client 依赖以下工件和它们的传递依赖项：

* org.elasticsearch.client:elasticsearch-rest-client
* org.elasticsearch:elasticsearch

#### 初始化

`RestHighLevelClient` 实例的构建需要一个 REST 低级客户端就像下面这样：
```java
RestHighLevelClient client =
    new RestHighLevelClient(lowLevelRestClient); //lowLevelRestClient注释
```

> lowLevelRestClient： 我们之前创建的[REST低级客户端](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-low-usage-initialization.html)实例

在本文档中关于Java高级客户端的的其余部分里，`RestHighLevelClient` 实例将以 `client` 被引用。

### 支持的 API

Java 高级 REST 客户端支持下列 API ：

单一文档 API

* [Index API](#Index-API)
* [Get API](#Get-API)
* [Delete API](#Delete-API)
* [Update API](#Update-API)

多文档 API

* [Bulk API](#Bulk-API)

搜索 API

* [Search API](#Search-API)
* [Search Scroll API](#Search-API)
* [Clear Scroll API](#Search-API)

杂项 API
* [Info API](#Info-API)

#### Index API

##### Index 请求

`IndexRequest` 要求下列参数：

```java
IndexRequest request = new IndexRequest(
        "posts",  //Index
        "doc",  //Type
        "1");   //Document id
String jsonString = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
        "}";
request.source(jsonString, XContentType.JSON); /以字符串提供的 Document source
```

##### 提供文档来源

文件来源可以以不同的方式提供：
```java
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("user", "kimchy");
jsonMap.put("postDate", new Date());
jsonMap.put("message", "trying out Elasticsearch");
IndexRequest indexRequest = new IndexRequest("posts", "doc", "1").source(jsonMap); //Map 作为文档源，它可以自动转换为 JSON 格式。
```

```java
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();
{
    builder.field("user", "kimchy");
    builder.field("postDate", new Date());
    builder.field("message", "trying out Elasticsearch");
}
builder.endObject();
IndexRequest indexRequest = new IndexRequest("posts", "doc", "1").source(builder); //XContentBuilder 对象作为文档源，由 Elasticsearch 内置的帮助器生成 JSON 内容
```

```java
IndexRequest indexRequest = new IndexRequest("posts", "doc", "1")
        .source("user", "kimchy",
                "postDate", new Date(),
                "message", "trying out Elasticsearch"); //以键值对对象作为文档来源，它自动转换为 JSON 格式
```

##### 可选参数

下列参数可选：
```java
request.routing("routing"); //Routing 值
```

```java
request.parent("parent"); //Parent 值
```

```java
request.timeout(TimeValue.timeValueSeconds(1)); //`TimeValue`类型的等待主分片可用的超时时间
request.timeout("1s"); //`String` 类型的等待主分片可用的超时时间
```

```java
request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL); //以 WriteRequest.RefreshPolicy 实例的刷新策略参数
request.setRefreshPolicy("wait_for"); // 字符串刷新策略参数
```

```java
request.version(2); //版本
```

```java
request.versionType(VersionType.EXTERNAL); //版本类型
```

```java
request.opType(DocWriteRequest.OpType.CREATE); //提供一个 DocWriteRequest.OpType 值作为操作类型
request.opType("create"); //字符串类型的操作类型参数: 可以是 create 或 update (默认值)
```

```java
request.setPipeline("pipeline"); //在索引文档之前要执行的摄取管道的名称
```

##### 同步执行

```Java
IndexResponse indexResponse = client.index(request);
```

##### 异步执行

```java
client.indexAsync(request, new ActionListener<IndexResponse>() {
    @Override
    public void onResponse(IndexResponse indexResponse) {
        //当操作成功完成的时候被调用。响应对象以参数的形式传入。
    }

    @Override
    public void onFailure(Exception e) {
        //故障时被调用。异常对象以参数的形式传入
    }
});
```

##### Index 响应

返回的“IndexResponse”可以检索有关执行操作的信息，如下所示：

```java
String index = indexResponse.getIndex();
String type = indexResponse.getType();
String id = indexResponse.getId();
long version = indexResponse.getVersion();
if (indexResponse.getResult() == DocWriteResponse.Result.CREATED) {
    //处理（如果需要）首次创建文档的情况
} else if (indexResponse.getResult() == DocWriteResponse.Result.UPDATED) {
    //处理（如果需要）文档已经存在时被覆盖的情况
}
ReplicationResponse.ShardInfo shardInfo = indexResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
    //处理成功碎片数少于总碎片的情况
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure : shardInfo.getFailures()) {
        String reason = failure.reason();//处理潜在的故障
    }
}
```

如果存在版本冲突，将抛出 `ElasticsearchException` :

```java
IndexRequest request = new IndexRequest("posts", "doc", "1")
        .source("field", "value")
        .version(1);
try {
    IndexResponse response = client.index(request);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        //表示是由于返回了版本冲突错误引发的异常
    }
}
```

发生同样的情况发生在`opType`设置为`create`但是已经存在具有相同索引，类型和id的文档时：

```java
IndexRequest request = new IndexRequest("posts", "doc", "1")
        .source("field", "value")
        .opType(DocWriteRequest.OpType.CREATE);
try {
    IndexResponse response = client.index(request);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        //表示由于返回了版本冲突错误引发的异常
    }
}
```

#### Get API

##### 获取请求对象
`GetRequest` 需要以下参数：
```
GetRequest getRequest = new GetRequest(
        "posts", // 索引
        "doc",  // 类别
        "1");   // 文档id
```

##### 可选参数

提供以下可选参数：

```java
request.fetchSourceContext(new FetchSourceContext(false)); //禁用检索源，默认为启用
```

```java
String[] includes = new String[]{"message", "*Date"};
String[] excludes = Strings.EMPTY_ARRAY;
FetchSourceContext fetchSourceContext = new FetchSourceContext(true, includes, excludes);
request.fetchSourceContext(fetchSourceContext); //设置源包含的特定域
```

```java
String[] includes = Strings.EMPTY_ARRAY;
String[] excludes = new String[]{"message"};
FetchSourceContext fetchSourceContext = new FetchSourceContext(true, includes, excludes);
request.fetchSourceContext(fetchSourceContext);


Configure source exclusion for specific fields
```

```java
request.storedFields("message");  //Configure retrieval for specific stored fields (requires fields to be stored separately in the mappings)
GetResponse getResponse = client.get(request);
String message = (String) getResponse.getField("message").getValue();  //Retrieve the message stored field (requires the field to be stored separately in the mappings)
```

```java
request.routing("routing");  //Routing value

request.parent("parent"); //Parent value

request.preference("preference"); //Preference value

request.realtime(false); //Set realtime flag to false (true by default)

request.refresh(true); //Perform a refresh before retrieving the document (false by default)

request.version(2); //Version

request.versionType(VersionType.EXTERNAL);  //Version type
```


##### 同步执行
```java
GetResponse getResponse = client.get(getRequest);
```
##### 异步执行
```java
client.getAsync(request, new ActionListener<GetResponse>() {
    @Override
    public void onResponse(GetResponse getResponse) {
//Called when the execution is successfully completed. The response is provided as an argument.
    }

    @Override
    public void onFailure(Exception e) {
//Called in case of failure. The raised exception is provided as an argument.
    }
});
```
##### 获取响应
The returned GetResponse allows to retrieve the requested document along with its metadata and eventually stored fields.
```java
String index = getResponse.getIndex();
String type = getResponse.getType();
String id = getResponse.getId();
if (getResponse.isExists()) {
    long version = getResponse.getVersion();
    String sourceAsString = getResponse.getSourceAsString(); //Retrieve the document as a String
    Map<String, Object> sourceAsMap = getResponse.getSourceAsMap(); //Retrieve the document as a Map<String, Object>
    byte[] sourceAsBytes = getResponse.getSourceAsBytes();  //Retrieve the document as a byte[]
} else {
//Handle the scenario where the document was not found. Note that although the returned response has 404 status code, a valid GetResponse is returned rather than an exception thrown. Such response does not hold any source document and its isExists method returns false.
}
```

当针对不存在的索引执行获取请求时，响应有404状态码，抛出一个 `ElasticsearchException` 异常，需要如下处理：
```java
GetRequest request = new GetRequest("does_not_exist", "doc", "1");
try {
    GetResponse getResponse = client.get(request);
} catch (ElasticsearchException e) {
    if (e.status() == RestStatus.NOT_FOUND) {
        // 处理因为索引不存在而抛出的异常，
    }
}
```
如果请求了特定文档版本，但现有文档具有不同的版本号，则会引发版本冲突：
```java
try {
    GetRequest request = new GetRequest("posts", "doc", "1").version(2);
    GetResponse getResponse = client.get(request);
} catch (ElasticsearchException exception) {
    if (exception.status() == RestStatus.CONFLICT) {
        //表示返回了版本冲突错误引发异常
    }
}
```

#### Delete API
#### Update API
#### Bulk API
#### Search API
#### Search Scroll API
#### Clear Scroll API
使用 Search Scroll API 的搜索上下文在超过滚动时间时，会自动清除。但是，建议当不再需要使用 Clear Scroll API 后尽可能快的释放搜索上下文。

##### Clear Scroll 请求
`ClearScrollRequest` 可以像如下方式创建:

```java
ClearScrollRequest request = new ClearScrollRequest();  // 创建一个新的 ClearScrollRequest
request.addScrollId(scrollId); // 添加一个滚动id到要清除的滚动标志列表里
```

https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-clear-scroll.html



#### Info API
##### 执行
集群信息可以通过 `info()` 方法被获取到：
```java
MainResponse response = client.info();
```
##### 响应
返回的 `MainResponse` 提供了有关集群的各种信息：
```java
ClusterName clusterName = response.getClusterName(); // 获取包含集群名称信息的 ClusterName 对象
String clusterUuid = response.getClusterUuid(); // 获取集群的唯一标识符
String nodeName = response.getNodeName(); // 获取执行请求的节点的名称
Version version = response.getVersion(); // 获取已执行请求的节点版本
Build build = response.getBuild(); // 获取已执行请求的节点的构建信息
```

### 使用 Java 建造者
Java高级REST客户端依赖于 Elasticsearch 核心项目提供的不同类型的 Java Builders 对象，包括：

**Query Builders**
查询构建器用于创建要在搜索请求中执行的查询。 查询DSL支持的每个类型的查询都有一个查询构建器。 每个查询构建器实现QueryBuilder接口，并允许为给定类型的查询设置特定选项。 创建后，可以将QueryBuilder对象设置为SearchSourceBuilder的查询参数。 “ 搜索请求”页面显示了如何使用SearchSourceBuilder和QueryBuilder对象构建完整搜索请求的QueryBuilder 。 “ 建筑物搜索查询”页面列出了所有可用的搜索查询及其相应的QueryBuilder对象和QueryBuilders辅助方法。
The query builders are used to create the query to execute within a search request. There is a query builder for every type of query supported by the Query DSL. Each query builder implements the QueryBuilder interface and allows to set the specific options for a given type of query. Once created, the QueryBuilder object can be set as the query parameter of SearchSourceBuilder. The Search Request page shows an example of how to build a full search request using SearchSourceBuilder and QueryBuilder objects. The Building Search Queries page gives a list of all available search queries with their corresponding QueryBuilder objects and QueryBuilders helper methods.

**Aggregation Builders**
与查询构建器类似，聚合构建器用于在搜索请求执行期间创建要计算的聚合。 对于Elasticsearch支持的每种类型的聚合（或管道聚合）都有一个聚合构建器。 所有构建器都扩展了AggregationBuilder类（或PipelineAggregationBuilder`class). Once created, `AggregationBuilder PipelineAggregationBuilder`class). Once created, `AggregationBuilder可以将PipelineAggregationBuilder`class). Once created, `AggregationBuilder对象设置为SearchSourceBuilder的聚合参数。 有一个例子，说明如何使用SearchSourceBuilder对象将AggregationBuilder对象定义为使用搜索请求页面中的搜索查询进行计算的聚合。 “ 构建聚合”页面提供了所有可用聚合以及其对应的AggregationBuilder对象和AggregationBuilders辅助方法的列表。
Similarly to query builders, the aggregation builders are used to create the aggregations to compute during a search request execution. There is an aggregation builder for every type of aggregation (or pipeline aggregation) supported by Elasticsearch. All builders extend the AggregationBuilder class (or PipelineAggregationBuilder`class). Once created, `AggregationBuilder objects can be set as the aggregation parameter of SearchSourceBuilder. There is a example of how AggregationBuilder objects are used with SearchSourceBuilder objects to define the aggregations to compute with a search query in Search Request page. The Building Aggregations page gives a list of all available aggregations with their corresponding AggregationBuilder objects and AggregationBuilders helper methods.

#### 构建查询
此页面列出了在 `QueryBuilders` 实用程序类中所有可用的搜索查询及其对应的QueryBuilder类名和帮助方法名称。
https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-query-builders.html

#### 构建聚合
https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-aggregation-builders.html

### 迁移指南

本节介绍如何将现有代码从 `TransportClient` 迁移到使用 Elasticsearch 5.6.0 版本发布的新的 Java 高级 REST 客户端。

> 译者注：本节不看了。。。原文见以下链接：
> https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-level-migration.html

### 许可证

Copyright 2013-2017 Elasticsearch

根据 Apache License, Version 2.0（“License”）许可; 您不得使用此文件，除非符合许可证。您可以获得许可证的副本

```
http://www.apache.org/licenses/LICENSE-2.0
```

除非由书面的法律要求或许可，根据许可证分发的软件以“按现状”基础分发，不附带任何明示或暗示的担保或条件。有关许可证下的权限和限制的特定语言，请参阅许可证。
