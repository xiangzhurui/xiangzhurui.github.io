---
layout: draf
title: Spring 3.2.x 无法再 jdk 1.8 上运行的解决方法
date: 2017-10-27 16:12:16
tags: [Spring, Java 8]
categories:
    -Spring
---
## 问题
在 Java 8 环境下启动使用 Spring 3.2.x 开发的应用时会抛出以下错误：
```
java.lang.RuntimeException: java.io.IOException: invalid constant type: 18
```

## 解决方法
问题原因为 javassist 包不兼容，解决方案为升级该依赖包版本。在Maven pom.xml 里配置如下版本即可：
```xml
<dependency>
    <groupId>org.javassist</groupId>
    <artifactId>javassist</artifactId>
    <version>3.22.0-GA</version>
</dependency>
```
> 以上配置会覆盖 由Spring 依赖传递引入的javassist，因为显示配置的依赖项优先级比依赖传递的依赖项更高，所以Maven自动解决依赖冲突时会选用显示配置的

## 其他

另外可能导致问题的原因在于 Java EE API 依赖范围不正确， `servlet-api` 的 `scope` 应为 `provided`。
如下所示：
* servlet 2.5 配置示例：
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
</dependency>
```
* servlet 3.0 配置示例：
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.0.1</version>
    <scope>provided</scope>
</dependency>
```
* servlet 3.1 配置示例：
```xml
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>javax.servlet-api</artifactId>
    <version>3.1.0</version>
    <scope>provided</scope>
</dependency>
```
