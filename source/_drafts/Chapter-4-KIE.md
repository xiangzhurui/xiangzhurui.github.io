---
title: Drools 7.3 第四章 KIE
tags: [Drools,Drools 7.3.0.Final,规则引擎,KIE]
---
# 4. KIE

KIE是Drools和jBPM的共享核心。它为构建，部署和使用资源提供了统一的方法论和编程模型。

## 目录
- [4.1. 概览]
  - [4.1.1. 项目剖析]
  - [4.1.2. 生命周期]
- [4.2. 构建，部署， 使用和运行]
  - [4.2.1. 介绍]
  - [4.2.2. 构建]
  - [4.2.3. 部署]
  - [4.2.4. 运行]
  - [4.2.5. 安装和部署电子表格]
  - [4.2.6. 构建，部署和使用示例]
- [4.3. 安全]
  - [4.3.1. 安全管理]

## 4.1. 概览
### 4.1.1. 项目剖析
### 4.1.2. 生命周期
使用KIE系统运作的不同方面或着生命周期，无论是Drools还是jBPM，通常都可以分为以下几种：
* Author
使用UI隐喻的创作知识(编写规则描述文件)，如：DRL，BPMN2，决策表，类模型。
* Build
  * 将创作的知识构建为可部署单元。
  * 对于KIE，这个单元部分是一个JAR。
* Test
在部署到应用程序之前测试KIE知识。
* Deploy
  * 将设备部署到应用程序可能利用的位置（消耗）。
  * KIE使用Maven风格存储库。
* Utilize
  * 加载JAR以提供KIE会话（KieSession），应用程序可以与之进行交互。
  * KIE通过KIE容器（KieContainer）在运行时公开了JAR。
  * KieSession是由KieContainer创建的，用于与运行时交互的KieSessions。
* Run
通过API与KieSession进行系统交互。
* Work
用户通过命令行或UI与KieSession进行交互。
* Manage
管理任何KieSession或KieContainer。


## 4.2. 构建，部署， 使用和运行
### 4.2.1. 介绍
6.0引入了一种新的配置和约定方法来构建知识库，而不是在5.x中使用编程构建器方法。建筑师仍然可以退缩，因为它用于工具集成。
### 4.2.2. 构建
*Figure 84. org.kie.api.core.builder* ![Figure 84. org.kie.api.core.builder](./builder.png)

*Example 22. 从类路径创建 KieContainer*

```java
KieServices kieServices = KieServices.Factory.get();
KieContainer kContainer = kieServices.getKieClasspathContainer();
```
“KieServices”是可以访问所有Kie建筑和运行时设施的接口：
### 4.2.3. 部署
### 4.2.4. 运行
### 4.2.5. 安装和部署电子表格
### 4.2.6. 构建，部署和使用示例
## 4.3. 安全
### 4.3.1. 安全管理
