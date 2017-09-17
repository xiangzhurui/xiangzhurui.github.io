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
update: 2017-09-17 12:41:49
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

```java
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
/**
*设置在同一请求进行多次尝试时应该遵守的超时时间。
* 默认值为30秒，与默认套接字超时相同。
* 如果自定义设置了套接字超时，则应该相应地调整最大重试超时。
*/
builder.setMaxRetryTimeoutMillis(10000);
```

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
            return requestConfigBuilder.setSocketTimeout(10000); // 备注：
        }
    });
```

> 设置允许修改默认请求配置的回调（例如：请求超时，认证，或者其他 [org.apache.http.client.config.RequestConfig.Builder](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/client/config/RequestConfig.Builder.html) 允许的设置）。

```java
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
    @Override
    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
        return httpClientBuilder.setProxy(new HttpHost("proxy", 9000, "http")); //备注：
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
关闭 `Sniffer` 非常重要，如此嗅探器后台线程才能正取关闭并释放他持有的资源。 `Sniffer` 对象应该与 `RestClient` 具有相同的生命周期，并在客户端之前关闭：

```java
sniffer.close();
restClient.close();
```
`Sniffer` 默认每5分钟更新一次节点列表。这个周期可以如下方式通过提供一个参数（毫秒数）自定义设置：

```java
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
Sniffer sniffer = Sniffer.builder(restClient)
        .setSniffIntervalMillis(60000).build();
```
也可以在发生故障是启用嗅探，这意味着每次故障后将直接获取并更新节点列表，而不是等到下一次正常的更新周期。此种情况时， `SniffOnFailureListener` 需要首先被创建，并将实例在 `RestClient` 创建时提供给它。 同样的，在之后创建 `Sniffer` 时，他需要被关联到同一个 `SniffOnFailureListener` 实例上，这个实例将在每个故障发生后被通知到，然后调用 `Sniffer` 去执行额外的嗅探行为。

```java
SniffOnFailureListener sniffOnFailureListener = new SniffOnFailureListener();
RestClient restClient = RestClient.builder(new HttpHost("localhost", 9200))
        .setFailureListener(sniffOnFailureListener) //为 RestClient 实例设置故障监听器
        .build();
Sniffer sniffer = Sniffer.builder(restClient
        .setSniffAfterFailureDelayMillis(30000) /* 故障后嗅探，不仅意味着每次故障后会更新节点，也会添加普通计划外的嗅探行为，默认情况是故障之后1分钟后，假设节点将恢复正常，那么我们希望尽可能快的获知。如上所述，周期可以通过 `setSniffAfterFailureDelayMillis` 方法在创建 Sniffer 实例时进行自定义设置。 需要注意的是，当没有启用故障监听时，这最后一个配置参数不会生效 */
        .build();
sniffOnFailureListener.setSniffer(sniffer); // 将 嗅探器关联到嗅探故障监听器上
```

`Elasticsearch` Nodes Info api 不会返回连接节点使用的协议，而只有他们的 `host:port` 键值对，因此默认使用 http。如果需要使用 `https` ，必须手动创建和提供 `ElasticsearchHostsSniffer` 实例，如下所示：

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
使用同样的方式，可以自定义设置 `sniffRequestTimeout`参数，该参数默认值为 1 秒。这是一个调用 Nodes Info api 时作为 querystring 参数的超时参数，这样当服务端超市时，仍然会返回一个有效响应，虽然它可能仅包含属于集群的一部分节点，其他节点会在随后响应。

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

同样的，一个自定义的 `HostsSniffer` 实现可以提供一个高级用法功能，比如可以从 Elasticsearch 之外的来源获取主机：
```java
RestClient restClient = RestClient.builder(
        new HttpHost("localhost", 9200, "http"))
        .build();
HostsSniffer hostsSniffer = new HostsSniffer() {
    @Override
    public List<HttpHost> sniffHosts() throws IOException {
        return null; // 从外部源获取主机
    }
};
Sniffer sniffer = Sniffer.builder(restClient)
        .setHostsSniffer(hostsSniffer).build();
```

### 许可证

Copyright 2013-2017 Elasticsearch

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

##### 文档来源

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

##### 删除请求对象
DeleteRequest 需要下列参数：
```java
DeleteRequest request = new DeleteRequest(
        "posts",    // index
        "doc",     //Type
        "1");  // Document id
```

##### 可选参数
提供下列可选参数：
```java
request.routing("routing");  // 路由值

request.parent("parent");  //Parent 值

request.timeout(TimeValue.timeValueMinutes(2)); // TimeValue 类型的等待主分片可用的超时时间
request.timeout("2m");  // 字符串类型的等待主分片可用的超时时间

request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);  // Refresh policy as a WriteRequest.RefreshPolicy instance
request.setRefreshPolicy("wait_for");  // Refresh policy as a String

request.version(2);   // Version

request.versionType(VersionType.EXTERNAL);  // Version type

```

##### 同步执行
```java
DeleteResponse deleteResponse = client.delete(request);
```

##### 异步执行
```java
client.deleteAsync(request, new ActionListener<DeleteResponse>() {
    @Override
    public void onResponse(DeleteResponse deleteResponse) {
        //Called when the execution is successfully completed. The response is provided as an argument
    }

    @Override
    public void onFailure(Exception e) {
        //Called in case of failure. The raised exception is provided as an argument
    }
});
```

##### 删除操作的响应
返回的 DeleteResponse 对象允许通过执行以下操作获取相关信息：

```java
String index = deleteResponse.getIndex();
String type = deleteResponse.getType();
String id = deleteResponse.getId();
long version = deleteResponse.getVersion();
ReplicationResponse.ShardInfo shardInfo = deleteResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
// Handle the situation where number of successful shards is less than total shards
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure : shardInfo.getFailures()) {
        String reason = failure.reason();  // Handle the potential failures
    }
}
```

也可以检查文档是否被发现：
```java
DeleteRequest request = new DeleteRequest("posts", "doc", "does_not_exist");
DeleteResponse deleteResponse = client.delete(request);
if (deleteResponse.getResult() == DocWriteResponse.Result.NOT_FOUND) {
    //如果被删除的文档没有被找到，做某些操作
}
```

如果有版本冲突，将抛出 ElasticsearchException 异常信息：
```java
try {
    DeleteRequest request = new DeleteRequest("posts", "doc", "1").version(2);
    DeleteResponse deleteResponse = client.delete(request);
} catch (ElasticsearchException exception) {
    if (exception.status() == RestStatus.CONFLICT) {
        //由于版本冲突错误导致的异常
    }
}
```

#### Update API

##### 更新请求

UpdateRequest 要求下列参数：
```java
UpdateRequest request = new UpdateRequest(
        "posts", // Index
        "doc",  // Type
        "1"); // Document id
```

Update API允许通过使用脚本或传递部分文档来更新现有文档。

##### 使用脚本更新
该脚本可以是一个内联脚本：
```java
Map<String, Object> parameters = singletonMap("count", 4); // 脚本参数以一个 Map 对象提供。

Script inline = new Script(ScriptType.INLINE, "painless", "ctx._source.field += params.count", parameters);  // 使用 painless 语言创建内联脚本，并传入参数值。
request.script(inline);  //  将脚本传递给更新请求对象
```

或者是一个存储脚本：
```java
// 引用一个名为 increment-field 的painless 的存储脚本
Script stored =
        new Script(ScriptType.STORED, "painless", "increment-field", parameters);   
request.script(stored); // 给请求设置脚本
```

##### 使用局部文档更新

使用局部文档更新时，局部文档将与现有文档合并。

局部文档可以以不同的方式提供：
```java
UpdateRequest request = new UpdateRequest("posts", "doc", "1");
String jsonString = "{" +
        "\"updated\":\"2017-01-01\"," +
        "\"reason\":\"daily update\"" +
        "}";
request.doc(jsonString, XContentType.JSON); // 以一个 JSON 格式的字符串提供的局部文档源
```

```
Map<String, Object> jsonMap = new HashMap<>();
jsonMap.put("updated", new Date());
jsonMap.put("reason", "daily update");
UpdateRequest request = new UpdateRequest("posts", "doc", "1")
        .doc(jsonMap); // 以一个可自动转换为 JSON 格式的 Map 提供局部文档源
```

```java
XContentBuilder builder = XContentFactory.jsonBuilder();
builder.startObject();
{
    builder.field("updated", new Date());
    builder.field("reason", "daily update");
}
builder.endObject();
UpdateRequest request = new UpdateRequest("posts", "doc", "1")
        .doc(builder); // 以 XContentBuilder 对象提供局部文档源， Elasticsearch 构建辅助器将生成 JSON 格式。
```

```java
UpdateRequest request = new UpdateRequest("posts", "doc", "1")
        .doc("updated", new Date(),
             "reason", "daily update"); // 以 Object 键值对提供局部文档源，它将被转换为 JSON 格式。
```

##### 更新或插入

如果文档不存在，可以使用upsert方法定义一些将作为新文档插入的内容：
```java
String jsonString = "{\"created\":\"2017-01-01\"}";
request.upsert(jsonString, XContentType.JSON);  // 以字符串提供更新插入的文档源
```
与局部文档更新类似，可以使用接受 `String`， `Map` ， `XContentBuilder` 或 `Object` 键值对的方式使用upsert 方法更新或插入文档的内容。

##### 可选参数
提供下列可选参数：
```java
request.routing("routing");  // 路由值

request.parent("parent");  //Parent 值

request.timeout(TimeValue.timeValueMinutes(2)); // TimeValue 类型的等待主分片可用的超时时间
request.timeout("2m");  // 字符串类型的等待主分片可用的超时时间

request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);  // Refresh policy as a WriteRequest.RefreshPolicy instance
request.setRefreshPolicy("wait_for");  // Refresh policy as a String

request.retryOnConflict(3);  //	How many times to retry the update operation if the document to update has been changed by another operation between the get and indexing phases of the update operation

request.fetchSource(true); //Enable source retrieval, disabled by default

request.version(2); // 版本号

request.detectNoop(false);  // Disable the noop detection

request.scriptedUpsert(true); // Indicate that the script must run regardless of whether the document exists or not, ie the script takes care of creating the document if it does not already exist.

request.docAsUpsert(true); // Indicate that the partial document must be used as the upsert document if it does not exist yet.

request.waitForActiveShards(2);  //Sets the number of shard copies that must be active before proceeding with the update operation.
request.waitForActiveShards(ActiveShardCount.ALL); //Number of shard copies provided as a ActiveShardCount: can be ActiveShardCount.ALL, ActiveShardCount.ONE or ActiveShardCount.DEFAULT (default)

```

```java
String[] includes = new String[]{"updated", "r*"};
String[] excludes = Strings.EMPTY_ARRAY;
request.fetchSource(new FetchSourceContext(true, includes, excludes)); // Configure source inclusion for specific fields
```

```java
String[] includes = Strings.EMPTY_ARRAY;
String[] excludes = new String[]{"updated"};
request.fetchSource(new FetchSourceContext(true, includes, excludes)); //Configure source exclusion for specific fields
```

##### 同步执行
```java
UpdateResponse updateResponse = client.update(request);
```
##### 异步执行
```java
client.updateAsync(request, new ActionListener<UpdateResponse>() {
    @Override
    public void onResponse(UpdateResponse updateResponse) {
        //	Called when the execution is successfully completed. The response is provided as an argument.
    }

    @Override
    public void onFailure(Exception e) {
        // Called in case of failure. The raised exception is provided as an argument.
    }
});
```
##### 更新响应
返回的UpdateResponse允许获取执行操作的相关信息，如下所示：
```java
String index = updateResponse.getIndex();
String type = updateResponse.getType();
String id = updateResponse.getId();
long version = updateResponse.getVersion();
if (updateResponse.getResult() == DocWriteResponse.Result.CREATED) {
//Handle the case where the document was created for the first time (upsert)
} else if (updateResponse.getResult() == DocWriteResponse.Result.UPDATED) {
//	Handle the case where the document was updated
} else if (updateResponse.getResult() == DocWriteResponse.Result.DELETED) {
//Handle the case where the document was deleted
} else if (updateResponse.getResult() == DocWriteResponse.Result.NOOP) {
//Handle the case where the document was not impacted by the update, ie no operation (noop) was executed on the document
}
```
当通过 `fetchSource` 方法在`UpdateRequest` 里设置了启用接收源，相应对象将包含被更新的文档源：
```java
GetResult result = updateResponse.getGetResult(); //Retrieve the updated document as a GetResult
if (result.isExists()) {
    String sourceAsString = result.sourceAsString(); //Retrieve the source of the updated document as a String
    Map<String, Object> sourceAsMap = result.sourceAsMap(); //Retrieve the source of the updated document as a Map<String, Object>
    byte[] sourceAsBytes = result.source(); //Retrieve the source of the updated document as a byte[]
} else {
    //Handle the scenario where the source of the document is not present in the response (this is the case by default)
}
```
这也可以用了检查分片故障：
```java
ReplicationResponse.ShardInfo shardInfo = updateResponse.getShardInfo();
if (shardInfo.getTotal() != shardInfo.getSuccessful()) {
    //Handle the situation where number of successful shards is less than total shards
}
if (shardInfo.getFailed() > 0) {
    for (ReplicationResponse.ShardInfo.Failure failure : shardInfo.getFailures()) {
        String reason = failure.reason(); //Handle the potential failures
    }
}
```
当对一个不存在的文档执行 `UpdateRequest ` 时，响应将包含 `404` 状态码，并抛出一个需要如下所示处理的 `ElasticsearchException` 异常：
```java
UpdateRequest request = new UpdateRequest("posts", "type", "does_not_exist").doc("field", "value");
try {
    UpdateResponse updateResponse = client.update(request);
} catch (ElasticsearchException e) {
    if (e.status() == RestStatus.NOT_FOUND) {
        // 处理由于文档不存在导致的异常。
    }
}
```
如果有文档版本冲突，也会抛出 `ElasticsearchException`:
```java
UpdateRequest request = new UpdateRequest("posts", "doc", "1")
        .doc("field", "value")
        .version(1);
try {
    UpdateResponse updateResponse = client.update(request);
} catch(ElasticsearchException e) {
    if (e.status() == RestStatus.CONFLICT) {
        // 表明异常是由于返回来了版本冲突错误导致的
    }
}
```

#### Bulk API

> **注意**： Java高级REST客户端提供[批量处理器](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-document-bulk.html#java-rest-high-document-bulk-processor)来协助大量请求

##### Bulk 请求
`BulkRequest` 可以被用在使用单个请求执行多个 `索引`，`更新` 和/或 `删除` 操作的情况下。

它要求至少要一个操作被添加到 Bulk 请求上：
```java
BulkRequest request = new BulkRequest();  // 创建 BulkRequest
request.add(new IndexRequest("posts", "doc", "1")  // 添加第一个 IndexRequest 到 Bulk 请求上， 参看 [Index API](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-document-index.html) 获取更多关于如何构建 IndexRequest 的信息.
        .source(XContentType.JSON,"field", "foo"));
request.add(new IndexRequest("posts", "doc", "2")   // 添加第二个 IndexRequest
        .source(XContentType.JSON,"field", "bar"));
request.add(new IndexRequest("posts", "doc", "3")  // 添加第三个 IndexRequest
        .source(XContentType.JSON,"field", "baz"));
```

> **警告**: Bulk API仅支持以JSON或SMILE编码的文档。 以任何其他格式提供文件将导致错误。

不同的操作类型也可以添加到同一个BulkRequest中：

```java
BulkRequest request = new BulkRequest();
request.add(new DeleteRequest("posts", "doc", "3")); //
Adds a DeleteRequest to the BulkRequest. See Delete API for more information on how to build DeleteRequest.
request.add(new UpdateRequest("posts", "doc", "2")
        .doc(XContentType.JSON,"other", "test")); //
Adds an UpdateRequest to the BulkRequest. See Update API for more information on how to build UpdateRequest.
request.add(new IndexRequest("posts", "doc", "4")   //Adds an IndexRequest using the SMILE format
        .source(XContentType.JSON,"field", "baz"));
```

##### 可选参数
提供下列可选参数：
```java
request.timeout(TimeValue.timeValueMinutes(2));
request.timeout("2m");


Timeout to wait for the bulk request to be performed as a TimeValue



Timeout to wait for the bulk request to be performed as a String

request.setRefreshPolicy(WriteRequest.RefreshPolicy.WAIT_UNTIL);
request.setRefreshPolicy("wait_for");                            


Refresh policy as a WriteRequest.RefreshPolicy instance



Refresh policy as a String

request.waitForActiveShards(2);
request.waitForActiveShards(ActiveShardCount.ALL);


Sets the number of shard copies that must be active before proceeding with the index/update/delete operations.



Number of shard copies provided as a ActiveShardCount: can be ActiveShardCount.ALL, ActiveShardCount.ONE or ActiveShardCount.DEFAULT (default)
```

##### 同步执行
```java
BulkResponse bulkResponse = client.bulk(request);
```

##### 异步执行
```java
client.bulkAsync(request, new ActionListener<BulkResponse>() {
    @Override
    public void onResponse(BulkResponse bulkResponse) {
        //Called when the execution is successfully completed. The response is provided as an argument and contains a list of individual results for each operation that was executed. Note that one or more operations might have failed while the others have been successfully executed.
    }

    @Override
    public void onFailure(Exception e) {
        //Called when the whole BulkRequest fails. In this case the raised exception is provided as an argument and no operation has been executed.
    }
});
```

##### Bulk 响应
返回的BulkResponse包含有关执行操作的信息，并允许对每个结果进行迭代，如下所示：
```java
for (BulkItemResponse bulkItemResponse : bulkResponse) { //迭代所有操作的结果
    DocWriteResponse itemResponse = bulkItemResponse.getResponse(); //Retrieve the response of the operation (successful or not), can be IndexResponse, UpdateResponse or DeleteResponse which can all be seen as DocWriteResponse instances

    if (bulkItemResponse.getOpType() == DocWriteRequest.OpType.INDEX
            || bulkItemResponse.getOpType() == DocWriteRequest.OpType.CREATE) {
                //	Handle the response of an index operation
        IndexResponse indexResponse = (IndexResponse) itemResponse;

    } else if (bulkItemResponse.getOpType() == DocWriteRequest.OpType.UPDATE) {
        //Handle the response of a update operation
        UpdateResponse updateResponse = (UpdateResponse) itemResponse;

    } else if (bulkItemResponse.getOpType() == DocWriteRequest.OpType.DELETE) {
        //	Handle the response of a delete operation
        DeleteResponse deleteResponse = (DeleteResponse) itemResponse;

    }
}
```

批量响应提供了一种快速检查一个或多个操作是否失败的方法：
```java
if (bulkResponse.hasFailures()) { // 只要有一个操作失败了，这个方法就返回 true

}
```

在这种情况下，有必要迭代所有运算结果，以检查操作是否失败，如果是，则检索相应的故障：
```java
for (BulkItemResponse bulkItemResponse : bulkResponse) {
    if (bulkItemResponse.isFailed()) { //Indicate if a given operation failed
        BulkItemResponse.Failure failure = bulkItemResponse.getFailure(); //Retrieve the failure of the failed operation

    }
}
```
##### Bulk 处理器
BulkProcessor通过提供允许索引/更新/删除操作在添加到处理器时透明执行的实用程序类来简化Bulk API的使用。

为了执行请求，BulkProcessor需要3个组件：

- `RestHighLevelClient`
这个客户端用来执行 BulkRequest 并接收 `BulkResponse` 。
- `BulkProcessor.Listener`
这个监听器会在每个 `BulkRequest` 执行之前和之后被调用，或者当 `BulkRequest` 失败时调用。
- `ThreadPool`
`BulkRequest`执行是使用这个池的线程完成的，允许`BulkProcessor`以非阻塞的方式工作，并允许在批量请求执行的同时接受新的索引/更新/删除请求。

然后 `BulkProcessor.Builder` 类可以被用来构建新的 `BulkProcessor` ：
```java
ThreadPool threadPool = new ThreadPool(settings); // 使用已有的 Settings 对象创建 ThreadPool。

BulkProcessor.Listener listener = new BulkProcessor.Listener() { // 创建 BulkProcessor.Listener
    @Override
    public void beforeBulk(long executionId, BulkRequest request) {
        //This method is called before each execution of a BulkRequest
    }

    @Override
    public void afterBulk(long executionId, BulkRequest request, BulkResponse response) {
        //	This method is called after each execution of a BulkRequest
    }

    @Override
    public void afterBulk(long executionId, BulkRequest request, Throwable failure) {
        //This method is called when a BulkRequest failed
    }
};

BulkProcessor bulkProcessor = new BulkProcessor.Builder(client::bulkAsync, listener, threadPool)
        .build(); // 通过调用 BulkProcessor.Builder 的 build() 方法创建 BulkProcessor。 RestHighLevelClient.bulkAsync() 将被用来执行 BulkRequest。
```

BulkProcessor.Builder 提供了方法来配置 BulkProcessor 应该如何处理请求的执行：

```java
BulkProcessor.Builder builder = new BulkProcessor.Builder(client::bulkAsync, listener, threadPool);
builder.setBulkActions(500);  //Set when to flush a new bulk request based on the number of actions currently added (defaults to 1000, use -1 to disable it)
builder.setBulkSize(new ByteSizeValue(1L, ByteSizeUnit.MB));  //	Set when to flush a new bulk request based on the size of actions currently added (defaults to 5Mb, use -1 to disable it)
builder.setConcurrentRequests(0); //Set the number of concurrent requests allowed to be executed (default to 1, use 0 to only allow the execution of a single request)
builder.setFlushInterval(TimeValue.timeValueSeconds(10L)); //	Set a flush interval flushing any BulkRequest pending if the interval passes (defaults to not set)
builder.setBackoffPolicy(BackoffPolicy.constantBackoff(TimeValue.timeValueSeconds(1L), 3)); //Set a constant back off policy that initially waits for 1 second and retries up to 3 times. See BackoffPolicy.noBackoff(), BackoffPolicy.constantBackoff() and BackoffPolicy.exponentialBackoff() for more options.
```

一旦创建了BulkProcessor，可以向其添加请求：
```java
IndexRequest one = new IndexRequest("posts", "doc", "1").
        source(XContentType.JSON, "title", "In which order are my Elasticsearch queries executed?");
IndexRequest two = new IndexRequest("posts", "doc", "2")
        .source(XContentType.JSON, "title", "Current status and upcoming changes in Elasticsearch");
IndexRequest three = new IndexRequest("posts", "doc", "3")
        .source(XContentType.JSON, "title", "The Future of Federated Search in Elasticsearch");

bulkProcessor.add(one);
bulkProcessor.add(two);
bulkProcessor.add(three);
```
这些请求将由 `BulkProcessor` 执行，它负责为每个批量请求调用 `BulkProcessor.Listener` 。
监听器提供方法接收 `BulkResponse` 和 `BulkResponse` ：

```java
BulkProcessor.Listener listener = new BulkProcessor.Listener() {
    @Override
    public void beforeBulk(long executionId, BulkRequest request) {
        int numberOfActions = request.numberOfActions(); //Called before each execution of a BulkRequest, this method allows to know the number of operations that are going to be executed within the BulkRequest
        logger.debug("Executing bulk [{}] with {} requests", executionId, numberOfActions);
    }

    @Override
    public void afterBulk(long executionId, BulkRequest request, BulkResponse response) {
        if (response.hasFailures()) {
            //在每个 BulkRequest 执行之后调用，此方法允许获知 BulkResponse 是否包含错误
            logger.warn("Bulk [{}] executed with failures", executionId);
        } else {
            logger.debug("Bulk [{}] completed in {} milliseconds", executionId, response.getTook().getMillis());
        }
    }

    @Override
    public void afterBulk(long executionId, BulkRequest request, Throwable failure) {
        logger.error("Failed to execute bulk", failure); //如果 BulkRequest 执行失败则调用，此方法可获知失败情况。
    }
};
```

一旦将所有请求都添加到BulkProcessor，其实例需要使用两种可用的关闭方法之一关闭。
一旦所有请求都被添加到了 `BulkProcessor`, 它的实例就需要使用两种可用的关闭方法之一进行关闭。

`awaitClose()` 可以被用来等待到所有请求都被处理，或者到指定的等待时间：
```java
boolean terminated = bulkProcessor.awaitClose(30L, TimeUnit.SECONDS);
```

如果所有批量请求完成，则该方法返回  `true` ，如果在完成所有批量请求之前等待时间过长，则返回 `false` 。

`close()` 方法可以被用来立即关闭 `BulkProcessor` ：
```java
bulkProcessor.close();
```
在关闭处理器之前，两个方法都会刷新已经被天教导处理器的请求，并禁止添加任何新的请求。

----

#### Search API

##### 搜索请求
SearchRequest用于与搜索文档，聚合，建议有关的任何操作，并且还提供了在生成的文档上请求突出显示的方法。
在最基本的形式中，我们可以向请求添加一个查询：
```java
SearchRequest searchRequest = new SearchRequest(); //穿件SeachRequest，Without arguments this runs against all indices.
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();// 大多数的搜索参数被添加到 SearchSourceBuilder 。它为每个进入请求体的每个东西都提供 setter 方法。
searchSourceBuilder.query(QueryBuilders.matchAllQuery());  // 添加一个 match_all 查询到 searchSourceBuilder 。
```

##### 可选参数
我们先看一下SearchRequest的一些可选参数：
```java
SearchRequest searchRequest = new SearchRequest("posts"); // 限制请求到某个索引上
searchRequest.types("doc");  // 限制请求的类别
```

还有一些其他有趣的可选参数：
```java
searchRequest.routing("routing");  // 设置路由参数、

searchRequest.indicesOptions(IndicesOptions.lenientExpandOpen()); // 设置IndicesOptions控制如何解析不可用索引以及扩展通配符表达式

searchRequest.preference("_local"); //使用首选参数，例如，执行搜索优先选择本地分片。 默认值是随机化分片。
```

##### 使用 SearchSourceBuilder

可以在SearchSourceBuilder上设置大多数控制搜索行为的选项，其中包含或多或少相当于 Rest API 的搜索请求正文中的选项。

以下是一些常见选项的示例：
```java
SearchSourceBuilder sourceBuilder = new SearchSourceBuilder(); //使用默认选项创建 SearchSourceBuilder 。
sourceBuilder.query(QueryBuilders.termQuery("user", "kimchy")); //设置查询对象。可以使任何类型的 QueryBuilder
sourceBuilder.from(0); //设置from选项，确定要开始搜索的结果索引。 默认为0。
sourceBuilder.size(5); //设置大小选项，确定要返回的搜索匹配数。 默认为10。
sourceBuilder.timeout(new TimeValue(60, TimeUnit.SECONDS)); //设置一个可选的超时时间，用于控制搜索允许的时间。

```
##### 构建查询
搜索查询可以使用 QueryBuilder 对象创建。 对于Elasticsearch的 [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/query-dsl.html) 支持的每个搜索查询类型，都存在QueryBuilder。
QueryBuilder 可以使用它的构造器创建：
```java
//创建一个字段“user”与文本“kimchy”相匹配的的全文匹配查询。
MatchQueryBuilder matchQueryBuilder = new MatchQueryBuilder("user", "kimchy");
```

创建后，QueryBuilder对象提供了配置搜索查询选项的方法：
```java
matchQueryBuilder.fuzziness(Fuzziness.AUTO); //在匹配查询上启用模糊匹配
matchQueryBuilder.prefixLength(3);  //在匹配查询上设置前缀长度
matchQueryBuilder.maxExpansions(10); //设置最大扩展选项以控制查询的模糊过程
```

`QueryBuilder` 对象也可以使用 `QueryBuilders` 工具类创建。这个类提供了使用链式编程风格的辅助方法来创建 `QueryBuilder` 对象：
```java
QueryBuilder matchQueryBuilder = QueryBuilders.matchQuery("user", "kimchy")
                                                .fuzziness(Fuzziness.AUTO)
                                                .prefixLength(3)
                                                .maxExpansions(10);
```
无论用于创建它的方法如何， `QueryBuilder` 对象必须添加到 `SearchSourceBuilder` 中，如下所示：
```java
searchSourceBuilder.query(matchQueryBuilder);
```

“[构建查询](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-query-builders.html)”页面列出了所有可用的搜索查询及其对应的QueryBuilder对象和QueryBuilders辅助方法。

##### 指定排序
`SearchSourceBuilder`允许添加一个或多个`SortBuilder`实例。 有四个特殊的实现（Field-，Score-，GeoDistance-和ScriptSortBuilder）。
```java
sourceBuilder.sort(new ScoreSortBuilder().order(SortOrder.DESC));  // 按_score降序排序（默认值）
sourceBuilder.sort(new FieldSortBuilder("_uid").order(SortOrder.ASC));  //也按_id字段排序升序
```

##### 源过滤
默认情况下，搜索请求返回文档的内容，`_source`但像 Rest API 中的内容一样，您可以覆盖此行为。例如，您可以完全关闭 `_source` 检索：
```java
sourceBuilder.fetchSource(false);
```
该方法还接受一个或多个通配符模式的数组，以便以更精细的方式控制哪些字段包含或排除：
```java
String[] includeFields = new String[] {"title", "user", "innerObject.*"};
String[] excludeFields = new String[] {"_type"};
sourceBuilder.fetchSource(includeFields, excludeFields);
```

##### 请求高亮
突出显示搜索结果可以通过设置来实现HighlightBuilder的 SearchSourceBuilder。可以通过向HighlightBuilder.Fielda 添加一个或多个实例来为每个字段定义不同的突出显示行为HighlightBuilder。
```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
HighlightBuilder highlightBuilder = new HighlightBuilder();//Creates a new HighlightBuilder
HighlightBuilder.Field highlightTitle =
        new HighlightBuilder.Field("title");//Create a field highlighter for the title field
highlightTitle.highlighterType("unified");  //Set the field highlighter type
highlightBuilder.field(highlightTitle);  //Add the field highlighter to the highlight builder
HighlightBuilder.Field highlightUser = new HighlightBuilder.Field("user");
highlightBuilder.field(highlightUser);
searchSourceBuilder.highlighter(highlightBuilder);
```
有很多选项，这在Rest API文档中有详细的介绍。Rest API参数（例如，`pre_tags`）通常由具有相似名称的 setter（例如#preTags（String ...））更改。

[随后](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html#java-rest-high-retrieve-highlighting)可以从 `SearchResponse` 中检索突出显示的文本片段。

##### 请求聚合
可以通过首先创建适当的集合AggregationBuilder然后在其上设置集合来将搜索添加到搜索结果中 SearchSourceBuilder。在下面的示例中，我们terms在公司名称上创建一个聚合，其中包含公司员工平均年龄的子聚合：
```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
TermsAggregationBuilder aggregation = AggregationBuilders.terms("by_company")
        .field("company.keyword");
aggregation.subAggregation(AggregationBuilders.avg("average_age")
        .field("age"));
searchSourceBuilder.aggregation(aggregation);
```
“ [构建聚合](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-aggregation-builders.html)”页面提供了所有可用聚合以及其相应AggregationBuilder对象和AggregationBuilders帮助方法的列表。

后面我们将看到如何[访问聚合](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html#java-rest-high-retrieve-aggs)的 SearchResponse。

##### 请求建议
要向搜索请求添加建议，请使用SuggestionBuilder从SuggestBuilders工厂类轻松访问的其中一个实现。建议构建者需要添加到顶层SuggestBuilder，本身可以设置在 顶层SearchSourceBuilder。
```java
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
SuggestionBuilder termSuggestionBuilder =
    SuggestBuilders.termSuggestion("user").text("kmichy"); //
Creates a new TermSuggestionBuilder for the user field and the text kmichy
SuggestBuilder suggestBuilder = new SuggestBuilder();
suggestBuilder.addSuggestion("suggest_user", termSuggestionBuilder); //Adds the suggestion builder and names it suggest_user
searchSourceBuilder.suggest(suggestBuilder);
```
后面我们将看到如何从 SearchResponse [获取建议](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html#java-rest-high-retrieve-suggestions)。
##### 自定义查询和聚合
该配置文件API可用于简档查询和聚集的执行针对特定搜索请求。为了使用它，配置文件标志必须设置为true SearchSourceBuilder：
```
SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
searchSourceBuilder.profile(true);
```
一旦SearchRequest执行，相应的SearchResponse将 [包含分析结果](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/java-rest-high-search.html#java-rest-high-retrieve-profile-results)。
##### 同步执行
SearchRequest以下列方式执行时，客户端SearchResponse在继续执行代码之前等待返回：
```java
SearchResponse searchResponse = client.search(searchRequest);
```

##### 异步执行
```java
client.searchAsync(searchRequest, new ActionListener<SearchResponse>() {
    @Override
    public void onResponse(SearchResponse searchResponse) {
        //执行成功完成时调用。
    }

    @Override
    public void onFailure(Exception e) {
        //当整个SearchRequest失败时调用。
    }
});
```

##### 搜索响应
通过执行搜索返回的 `SearchResponse` 提供了关于搜索执行本身以及对返回的文档的访问的详细信息。 首先，有关于请求执行本身的有用信息，如HTTP状态代码，执行时间或请求提前终止或超时：

```java
RestStatus status = searchResponse.status();
TimeValue took = searchResponse.getTook();
Boolean terminatedEarly = searchResponse.isTerminatedEarly();
boolean timedOut = searchResponse.isTimedOut();
```
其次，响应还通过提供关于搜索影响的分片总数以及成功与不成功的分片的统计信息，提供关于分片级别执行的信息。 可能的故障也可以通过遍历ShardSearchFailures上的数组进行迭代来处理，如下例所示：
```java
int totalShards = searchResponse.getTotalShards();
int successfulShards = searchResponse.getSuccessfulShards();
int failedShards = searchResponse.getFailedShards();
for (ShardSearchFailure failure : searchResponse.getShardFailures()) {
    // 故障应该在这里被处理
}
```

##### 检索 SearchHits

要访问返回的文档，我们需要首先得到响应中包含的 `SearchHits` ：
```java
SearchHits hits = searchResponse.getHits();
```
将SearchHits提供命中的所有全局信息，比如命中总数或最大分数：
```java
long totalHits = hits.getTotalHits();
float maxScore = hits.getMaxScore();
```
嵌套在 `SearchHits` 的各个搜索结果可以被迭代访问：
```java
SearchHit[] searchHits = hits.getHits();
for (SearchHit hit : searchHits) {
    // do something with the SearchHit
}
```
SearchHit可以访问基本信息，如索引，类型，文档ID 以及每个搜索匹配的分数：
```java
String index = hit.getIndex();
String type = hit.getType();
String id = hit.getId();
float score = hit.getScore();
```
此外，它可以让您将文档源作为简单的JSON-String或键/值对的映射。 在该映射中，字段名为键，含字段值为值。 多值字段作为对象的列表返回，嵌套对象作为另一个键/值映射。 这些情况需要相应地执行：
```java
String sourceAsString = hit.getSourceAsString();
Map<String, Object> sourceAsMap = hit.getSourceAsMap();
String documentTitle = (String) sourceAsMap.get("title");
List<Object> users = (List<Object>) sourceAsMap.get("user");
Map<String, Object> innerObject = (Map<String, Object>) sourceAsMap.get("innerObject");
```
##### Retrieving Highlighting
如果请求，可以从SearchHit结果中的每一个中检索突出显示的文本片段。命中对象提供对HighlightField实例的字段名称映射的访问，每个实例都包含一个或多个突出显示的文本片段：

```java
SearchHits hits = searchResponse.getHits();
for (SearchHit hit : hits.getHits()) {
    Map<String, HighlightField> highlightFields = hit.getHighlightFields();
    HighlightField highlight = highlightFields.get("title"); //获取该title领域 的突出显示
    Text[] fragments = highlight.fragments();  //获取包含突出显示的字段内容的一个或多个片段
    String fragmentString = fragments[0].string();
}
```

##### 检索聚合
可以SearchResponse通过首先获取聚合树的根，Aggregations对象，然后通过名称获取聚合来检索聚合。

```java
Aggregations aggregations = searchResponse.getAggregations();
Terms byCompanyAggregation = aggregations.get("by_company"); //Get the by_company terms aggregation

Bucket elasticBucket = byCompanyAggregation.getBucketByKey("Elastic"); //
Avg averageAge = elasticBucket.getAggregations().get("average_age"); //Get the average_age sub-aggregation from that bucket
double avg = averageAge.getValue();
```

请注意，如果按名称访问聚合，则需要根据您请求的聚合类型指定聚合接口，否则将抛出 `ClassCastException` ：

```java
//This will throw an exception because "by_company" is a terms aggregation but we try to retrieve it as a range aggregation
Range range = aggregations.get("by_company");
```

也可以以聚合名称键入的映射来访问所有聚合。在这种情况下，转换为适当聚合接口需要明确发生：
```java
Map<String, Aggregation> aggregationMap = aggregations.getAsMap();
Terms companyAggregation = (Terms) aggregationMap.get("by_company");
```
还有getter 方法将所有顶级聚合作为列表返回：
```java
List<Aggregation> aggregationList = aggregations.asList();
```
最后但并非最不重要的是，您可以迭代所有聚合，然后根据其类型决定如何进一步处理它们：
```java
for (Aggregation agg : aggregations) {
    String type = agg.getType();
    if (type.equals(TermsAggregationBuilder.NAME)) {
        Bucket elasticBucket = ((Terms) agg).getBucketByKey("Elastic");
        long numberOfDocs = elasticBucket.getDocCount();
    }
}
```
##### 检索建议
要从SearchResponse返回建议，请使用Suggest对象作为入口点，然后检索嵌套的建议对象：
```java
Suggest suggest = searchResponse.getSuggest();// Use the Suggest class to access suggestions
TermSuggestion termSuggestion = suggest.getSuggestion("suggest_user");// Suggestions can be retrieved by name. You need to assign them to the correct type of Suggestion class (here TermSuggestion), otherwise a ClassCastException is thrown
for (TermSuggestion.Entry entry : termSuggestion.getEntries()) {// Iterate over the suggestion entries
    for (TermSuggestion.Entry.Option option : entry) {// Iterate over the options in one entry
        String suggestText = option.getText().string();
    }
}
```

##### 检索分析结果
从SearchResponse使用该getProfileResults()方法检索分析结果。此方法返回Map包含执行中ProfileShardResult涉及的每个分片的对象 SearchRequest。ProfileShardResult存储在Map使用唯一地标识分析结果对应的分片的密钥。

这是一个示例代码，显示如何遍历每个分片的所有分析结果：
```java
Map<String, ProfileShardResult> profilingResults = searchResponse.getProfileResults(); //Retrieve the Map of ProfileShardResult from the SearchResponse
for (Map.Entry<String, ProfileShardResult> profilingResult : profilingResults.entrySet()) {  //Profiling results can be retrieved by shard’s key if the key is known, otherwise it might be simpler to iterate over all the profiling results
    String key = profilingResult.getKey();//Retrieve the key that identifies which shard the ProfileShardResult belongs to
    ProfileShardResult profileShardResult = profilingResult.getValue();//Retrieve the ProfileShardResult for the given shard
}
```

ProfileShardResult对象本身包含一个或多个查询配置文件结果，一个针对基本的Lucene索引执行的每个查询：

```java
List<QueryProfileShardResult> queryProfileShardResults = profileShardResult.getQueryProfileResults(); // 检索 QueryProfileShardResult 列表
for (QueryProfileShardResult queryProfileResult : queryProfileShardResults) {
//迭代每个 QueryProfileShardResult
}
```

每个都QueryProfileShardResult可以访问详细的查询树执行，作为ProfileResult对象列表返回 ：
```java
for (ProfileResult profileResult : queryProfileResult.getQueryResults()) {//迭代配置文件结果
    String queryName = profileResult.getQueryName(); //检索Lucene查询的名称
    long queryTimeInMillis = profileResult.getTime(); //检索在执行Lucene查询时花费的时间
    List<ProfileResult> profiledChildren = profileResult.getProfiledChildren();//
检索子查询的配置文件结果（如果有）
}
```

Rest API文档包含有关[查询分析信息](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/_profiling_queries.html#_literal_query_literal_section)描述的[Profiling Queries](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/_profiling_queries.html)的更多[信息](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/_profiling_queries.html#_literal_query_literal_section)。

该 `QueryProfileShardResult` 还可以访问了Lucene的收藏家的分析信息：
```java
CollectorResult collectorResult = queryProfileResult.getCollectorResult();  //检索Lucene收集器的分析结果
String collectorName = collectorResult.getName();  //检索Lucene 的名字
Long collectorTimeInMillis = collectorResult.getTime();//以毫秒计算的时间用于执行Lucene集合
List<CollectorResult> profiledChildren = collectorResult.getProfiledChildren();//
检索子集合的个人资料结果（如果有）
```

Rest API文档包含有关[Lucene收集器](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/_profiling_queries.html#_literal_collectors_literal_section)的分析信息的更多信息 。

以与查询树执行非常相似的方式，QueryProfileShardResult对象可以访问详细的聚合树执行：
```java
AggregationProfileShardResult aggsProfileResults = profileShardResult.getAggregationProfileResults(); // 检索 AggregationProfileShardResult
for (ProfileResult profileResult : aggsProfileResults.getProfileResults()) { //迭代聚合配置文件结果
    String aggName = profileResult.getQueryName(); //Retrieve the type of the aggregation (corresponds to Java class used to execute the aggregation)
    long aggTimeInMillis = profileResult.getTime();//Retrieve the time in millis spent executing the Lucene collector
    List<ProfileResult> profiledChildren = profileResult.getProfiledChildren(); //Retrieve the profile results for the sub-aggregations (if any)
}
```
Rest API文档包含更多关关于 [Profiling Aggregations](https://www.elastic.co/guide/en/elasticsearch/reference/5.6/_profiling_aggregations.html) 的信息

#### Search Scroll API

Scroll API可用于从搜索请求中检索大数量的结果。

为了使用滚动，需要按照给定的顺序执行以下步骤。

##### 初始化搜索滚动上下文
##### 检索所有相关文档
##### 清除滚动上下文
##### 可选参数
##### 同步执行
##### 异步执行
##### 响应
##### 完整示例


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
    The query builders are used to create the query to execute within a search request. There is a query builder for every type of query supported by the Query DSL. Each query builder implements the QueryBuilder interface and allows to set the specific options for a given type of query. Once created, the QueryBuilder object can be set as the query parameter of SearchSourceBuilder. The Search Request page shows an example of how to build a full search request using SearchSourceBuilder and QueryBuilder objects. The Building Search Queries page gives a list of all available search queries with their corresponding QueryBuilder objects and QueryBuilders helper methods.
**Aggregation Builders**
    Similarly to query builders, the aggregation builders are used to create the aggregations to compute during a search request execution. There is an aggregation builder for every type of aggregation (or pipeline aggregation) supported by Elasticsearch. All builders extend the AggregationBuilder class (or `PipelineAggregationBuilder`class). Once created, `AggregationBuilder objects can be set as the aggregation parameter of SearchSourceBuilder. There is a example of how AggregationBuilder objects are used with SearchSourceBuilder objects to define the aggregations to compute with a search query in Search Request page. The Building Aggregations page gives a list of all available aggregations with their corresponding AggregationBuilder objects and AggregationBuilders helper methods.

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
