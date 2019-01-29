---
title: Apache Kafka 起步
categories: MQ
tags: Apache Kafka
date: 2019-01-29 13:41:31
---

> Apache Kafka 安装、基本配置和启动

## Apache Kafka 服务安装分一下两步走：

1. 启动zookeeper服务
2. 启动kafka服务

## 安装命令

### 启动 zookeeper 服务
> 若设备上没有zookeeper单独安装的话，可以使用与kafka一起打包的脚本来启动，
> 本人设备上已有zookeeper， 不在单独启动。

首先，命令行切换到kafka根目录，然后依次运行一下命令：

```shell
## Windows下
.\bin\windows\zookeeper-server-start ./config/zookeeper.properties
## mac & linux
.\bin\kafka-server-start.sh .\config\zookeeper.properties
```

```shell
## Windows 下
.\bin\windows\kafka-server-start.bat .\config\server.properties
## macOS & Linux 下
./bin/kafka-server-start.sh ./config/server.properties
```

## 配置修改
1. 防火墙放开zookeeper（默认2181） 和kafka端口号（默认9092）
2. kafka 配置文件 `server.properties` 修改,在改配置文件中添加以下两项：

```properties
listeners=PLAINTEXT://:9092
## your.host.name 修改为域名或ip
advertised.listeners=PLAINTEXT://your.host.name:9092
```

> 注：若客户端应用启动时出现以下报错，则可能即是以上`server.properties`配置的问题

```java
2019-01-17 11:50:54.500 ERROR 29244 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.context.ApplicationContextException: Failed to start bean 'org.springframework.kafka.config.internalKafkaListenerEndpointRegistry'; nested exception is org.apache.kafka.common.errors.TimeoutException: Timeout expired while fetching topic metadata
	at org.springframework.context.support.DefaultLifecycleProcessor.doStart(DefaultLifecycleProcessor.java:185) ~[spring-context-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.context.support.DefaultLifecycleProcessor.access$200(DefaultLifecycleProcessor.java:53) ~[spring-context-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.context.support.DefaultLifecycleProcessor$LifecycleGroup.start(DefaultLifecycleProcessor.java:360) ~[spring-context-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.context.support.DefaultLifecycleProcessor.startBeans(DefaultLifecycleProcessor.java:158) ~[spring-context-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.context.support.DefaultLifecycleProcessor.onRefresh(DefaultLifecycleProcessor.java:122) ~[spring-context-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.finishRefresh(AbstractApplicationContext.java:879) ~[spring-context-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.finishRefresh(ServletWebServerApplicationContext.java:161) ~[spring-boot-2.1.0.RELEASE.jar:2.1.0.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:549) ~[spring-context-5.1.2.RELEASE.jar:5.1.2.RELEASE]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:140) ~[spring-boot-2.1.0.RELEASE.jar:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:775) [spring-boot-2.1.0.RELEASE.jar:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:397) [spring-boot-2.1.0.RELEASE.jar:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:316) [spring-boot-2.1.0.RELEASE.jar:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1260) [spring-boot-2.1.0.RELEASE.jar:2.1.0.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1248) [spring-boot-2.1.0.RELEASE.jar:2.1.0.RELEASE]
	at com.example.Sample01Application.main(Sample01Application.java:46) [classes/:na]
Caused by: org.apache.kafka.common.errors.TimeoutException: Timeout expired while fetching topic metadata
```
