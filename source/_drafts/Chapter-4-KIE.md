---
title: Drools 7.3.0.Final 文档 - 第4章 KIE
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
#### 4.2.2.2. `kmodule.xml` 文件
#### 4.2.2.4. 以编程方式定义 KieModule

也可以以编程方式定义属于 `KieModule` 的 `KieBase` 们和 `KieSession` 们，而不是在 `kmodule.xml` 文件中声明定义。相同的编程API还允许明确添加包含Kie工件的文件，而不是从项目的资源文件夹自动读取它们。要做到这一点，有必要创建一种 `KieFileSystem` 虚拟文件系统，并将项目中包含的所有资源添加到它。

![Figure 89. KieFileSystem](./Chapter-4-KIE/BuildDeployUtilizeAndRun/KieFileSystem.png)
*Figure 89. KieFileSystem*
像所有其他的纪伊核心部件，你可以从 `KieServices` 得到的一个 `KieFileSystem` 实例。`kmodule.xml` 配置文件必须被添加到文件系统中。这是个强制性步骤。Kie 还提供了一个由 `KieModuleModel` 程序实现的方便流式风格的 API，，以便以编程方式创建此文件。
![Figure 90. KieModuleModel](./Chapter-4-KIE/BuildDeployUtilizeAndRun/KieModuleModel.png)
*Figure 90. KieModuleModel*
在实际中使用它需要从 `KieServices` 穿件一个 `KieModuleModel` ， 并配置所需的 `KieBases` 和 `KieSessions` ， 并将其转换成 XML 并添加到 `KieFileSystem` 里。 这个下面这个示例程序展示了这一点：
* 示例28.以编程方式创建kmodule.xml并将其添加到KieFileSystem
```java
KieServices kieServices = KieServices.Factory.get();
KieModuleModel kieModuleModel = kieServices.newKieModuleModel();

KieBaseModel kieBaseModel1 = kieModuleModel.newKieBaseModel( "KBase1 ")
        .setDefault( true )
        .setEqualsBehavior( EqualityBehaviorOption.EQUALITY )
        .setEventProcessingMode( EventProcessingOption.STREAM );

KieSessionModel ksessionModel1 = kieBaseModel1.newKieSessionModel( "KSession1" )
        .setDefault( true )
        .setType( KieSessionModel.KieSessionType.STATEFUL )
        .setClockType( ClockTypeOption.get("realtime") );

KieFileSystem kfs = kieServices.newKieFileSystem();
kfs.writeKModuleXML(kieModuleModel.toXML());
```
在这一点上，还需要KieFileSystem通过流畅的API 增加构成项目的所有其他Kie工件。这些工件必须添加在相应的通常的Maven项目的相同位置。
* 示例29.将Kie工件添加到KieFileSystem
```java
KieFileSystem kfs = ...
kfs.write( "src/main/resources/KBase1/ruleSet1.drl", stringContainingAValidDRL )
        .write( "src/main/resources/dtable.xls",
                kieServices.getResources().newInputStreamResource( dtableFileStream ) );
```
* 图 92. KieBuilder
当 `KieFileSystem` 的内容成功构建时，结果 `KieModule` 将自动被添加到 `KieRepository` 。 `KieRepository` 是一个单例，作为所有可用 `KieModule` 的仓库。

* 示例31.构建KieFileSystem的内容并创建KieContainer
```java
KieServices kieServices = KieServices.Factory.get();
KieFileSystem kfs = ...
kieServices.newKieBuilder(kfs).buildAll();
KieContainer kieContainer = kieServices.newKieContainer(kieServices.getRepository().getDefaultReleaseId());
```
* 示例32.检查编译没有产生任何错误
```java
KieBuilder kieBuilder = kieServices.newKieBuilder( kfs ).buildAll();
assertEquals( 0, kieBuilder.getResults().getMessages( Message.Level.ERROR ).size() );
```


### 4.2.3. 部署
#### 4.2.3.1. KieBase

`KieBase` 是所有应用知识定义的一个仓库。它包含规则，进程，函数和类型模型。


### 4.2.4. 运行
### 4.2.5. 安装和部署电子表格
### 4.2.6. 构建，部署和使用示例
## 4.3. 安全
### 4.3.1. 安全管理
