---
title: elasticsearch Java rest client 配置Spring bean
date: 2017-08-21 20:00:00 00:00:00
tags: [Elasticsearch,Spring, 配置]
---

> 一般而言类似数据源的对象我们会配置一个全局可用的bean，本文简述elasticsearch 5.5 java restClient在 Spring 里的配置(单点es)

## RestClient 的Maven 依赖(Spring 相关依赖略)
```xml
<dependency>
    <groupId>org.elasticsearch.client</groupId>
    <artifactId>rest</artifactId>
    <version>${elasticsearch.version}</version>
</dependency>
```

## Java config 配置方式

```java
import org.elasticsearch.client.RestClient;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class EsConfig {
    @Value("es.host")
    private String host;

    @Value("es.port")
    private String port;

    @Value("es.scheme")
    private String scheme;

    private HttpHost[] hostList;


    public EsConfig() {
    }


    @Bean(name = "restClient", destroyMethod = "close")
    public RestClient getRestClient() {
        return RestClient.builder(hostList).build();
    }
}
```

## XML 配置方式
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="restClientBuilder" class="org.elasticsearch.client.RestClientBuilder">
        <constructor-arg ref="httpHost"/>
    </bean>
    <bean id="restClient" class="org.elasticsearch.client.RestClient" factory-bean="restClientBuilder"
          factory-method="build" destroy-method="close">
    </bean>
    <bean id="httpHost" class="org.apache.http.HttpHost">
        <constructor-arg value="localhost"/>
        <constructor-arg value="9200"/>
        <constructor-arg value="http"/>
    </bean>
</beans>
```
