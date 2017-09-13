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

```
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
```
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
Header[] defaultHeaders = new Header[]{new BasicHeader("header", "value")};
builder.setDefaultHeaders(defaultHeaders);
```

> 设置每个请求发送时的默认头文件，以免每个请求都必须指定它们。

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

    }
});
```
>设置每次节点发生故障时收到通知的侦听器，触发如果需要采取的行动。内部嗅探到故障时被启用。

```java
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setRequestConfigCallback(new RestClientBuilder.RequestConfigCallback() {
    @Override
    public RequestConfig.Builder customizeRequestConfig(RequestConfig.Builder requestConfigBuilder) {
        return requestConfigBuilder.setSocketTimeout(10000);
    }
});
```

> 设置允许修改默认请求配置的回调（例如：请求超时，认证，或者其他 [org.apache.http.client.config.RequestConfig.Builder](https://hc.apache.org/httpcomponents-client-ga/httpclient/apidocs/org/apache/http/client/config/RequestConfig.Builder.html) 允许的设置）。

```java
RestClientBuilder builder = RestClient.builder(new HttpHost("localhost", 9200, "http"));
builder.setHttpClientConfigCallback(new RestClientBuilder.HttpClientConfigCallback() {
    @Override
    public HttpAsyncClientBuilder customizeHttpClient(HttpAsyncClientBuilder httpClientBuilder) {
        return httpClientBuilder.setProxy(new HttpHost("proxy", 9000, "http"));
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
#### Javadoc
#### Maven 仓库

REST 客户端嗅探器与 elasticsearch 的发行周期相同。可以使用期望的嗅探器版本替换，但必须是 `5.0.0-alpha4` 之后的版本。嗅探器版本与其通信的 Elasticsearch 版本之间没有关联。嗅探器支持从 `elasticsearch 2.x` 及以上的版本上获取节点列表。

##### Maven 配置
Here is how you can configure the dependency using maven as a dependency manager. Add the following to your pom.xml file:
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>elasticsearch-rest-client-sniffer</artifactId>
    <version>5.6.0</version>
</dependency>
```
##### Gradle 配置
Here is how you can configure the dependency using gradle as a dependency manager. Add the following to your build.gradle file:
```
dependencies {
    compile 'org.elasticsearch.client:elasticsearch-rest-client-sniffer:5.6.0'
}
```
#### 用法

### 许可证

## Java 高级 REST 客户端
Java高级REST客户端可以在Java Low Level REST客户机之上工作。 其主要目标是公开特定方法的API，接受请求对象作为参数并返回响应对象，以便客户端自己处理请求编组和响应解组。

每个API可以同步或异步地调用。 同步方法返回一个响应对象，而名称以异步后缀结尾的异步方法需要收到响应或错误后才会通知（在低级别客户端管理的线程池上）的侦听器参数。

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
REST 高级客户端的 javadoc 可以在这里找到[https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-high-level-client/5.6.0/index.html]( https://artifacts.elastic.co/javadoc/org/elasticsearch/client/elasticsearch-rest-high-level-client/5.6.0/index.html)

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

```
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
    new RestHighLevelClient(lowLevelRestClient);
```

> 我们通过REST低级客户端实例创建它

在本文档中关于Java高级客户端的的其余部分里，`RestHighLevelClient` 实例将作为 `client` 被引用。

### 支持的 API
Java 高级 REST 客户端支持以下 API ：
单一文档API
* Index API
* Get API
* Delete API
* Update API
多文档API
* Bulk API
搜索API
* Search API
* Search Scroll API
* Clear Scroll API
杂项API
* Info API

#### Index API
##### Index Request
`IndexRequest` 要求下列参数：
```java
IndexRequest request = new IndexRequest(
        "posts",  //1
        "doc",  //2
        "1");   //3
String jsonString = "{" +
        "\"user\":\"kimchy\"," +
        "\"postDate\":\"2013-01-30\"," +
        "\"message\":\"trying out Elasticsearch\"" +
        "}";
request.source(jsonString, XContentType.JSON); //4
```

1. Index
2. Type
3. Document id
4. Document source provided as a String

#### Get API
#### Delete API
#### Update API
#### Bulk API
#### Search API
#### Search Scroll API
#### Clear Scroll API
#### Info API

### 使用 Java 建造者
The Java High Level REST Client depends on the Elasticsearch core project which provides different types of Java Builders objects, including:

**Query Builders**
The query builders are used to create the query to execute within a search request. There is a query builder for every type of query supported by the Query DSL. Each query builder implements the QueryBuilder interface and allows to set the specific options for a given type of query. Once created, the QueryBuilder object can be set as the query parameter of SearchSourceBuilder. The Search Request page shows an example of how to build a full search request using SearchSourceBuilder and QueryBuilder objects. The Building Search Queries page gives a list of all available search queries with their corresponding QueryBuilder objects and QueryBuilders helper methods.

**Aggregation Builders**
Similarly to query builders, the aggregation builders are used to create the aggregations to compute during a search request execution. There is an aggregation builder for every type of aggregation (or pipeline aggregation) supported by Elasticsearch. All builders extend the AggregationBuilder class (or PipelineAggregationBuilder`class). Once created, `AggregationBuilder objects can be set as the aggregation parameter of SearchSourceBuilder. There is a example of how AggregationBuilder objects are used with SearchSourceBuilder objects to define the aggregations to compute with a search query in Search Request page. The Building Aggregations page gives a list of all available aggregations with their corresponding AggregationBuilder objects and AggregationBuilders helper methods.

#### 构建查询
此页面列出了在 `QueryBuilders` 实用程序类中所有可用的搜索查询及其对应的QueryBuilder类名和帮助方法名称。
#### 构建聚合

### 迁移指南
### 许可证
