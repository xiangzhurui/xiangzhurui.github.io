---
title: KIE API 介绍
categories:
  - Drools
tags:
  - Drools
  - KIE
  - API
  - 规则引擎
date: 2017-10-04 20:25:12
---

## Maven 依赖
`kie-api` 版本号与 Drools 版本一直。
```xml
<dependency>
    <groupId>org.kie</groupId>
    <artifactId>kie-api</artifactId>
    <version>${drools.version}</version>
</dependency>
```
## KIE 类

* `KieServices`
  KIE 整体的入口,可以用来创建 `KieContainer` , `KieResources` , `KieFileSystem` 等

* `KieContainer`
  `KieContainer` 就是一个 `KieBase` 的容器，可以根据 `kmodule.xml` 里描述的 `KieBase` 信息来获取具体的 `KieSession`

* `KieBase`
  `KieBase` 就是一个知识仓库，包含了若干的规则、流程、方法等，在Drools中主要就是规则和方法， `KieBase` 本身并不包含运行时的数据之类的，如果需要执行规则 `KieBase` 中的规则的话，就需要根据 `KieBase` 创建 `KieSession`

* `KieSession`
  `KieSession` 就是一个跟Drools引擎打交道的会话，其基于KieBase创建，它会包含运行时数据，包含“事实 Fact”，并对运行时数据事实进行规则运算

* `KieModule`
  是一个包含了多个kiebase定义的容器。一般用kmodule.xml来表示

* `KieModuleModel`
  是 `kmodule.xml` 文件的java表示，可以不用添加xml文件而是通过程序代码的方式来构建，

* `KieFileSystem`
  一个完整的文件系统,包括资源和组织结构

* `KieBuilder`
  当把所有的规则文件添加到 `KieFileSystem` 中后，通过把 `KieFileSystem` 传递给一个 `KieBuilder` ，可以构建出这个虚拟文件系统。其中有个 `buildAll()` 方法，会在构建好虚拟文件系统后，自动去构建 `KieModule`

* `KieRepository`
  是一个 `KieModule` 的仓库，包含了所有的 `KieModule` 描述，用一个 `ReleaseId` 做区分

* `KieResources`
  是一个定义了如何获取资源的工厂，包括 url，classpath，filesystem 等

* `kiemodule.xml` 结构如下所示：

    ```xml
    <kmodule xmlns="http://jboss.org/kie/6.0.0/kmodule">
        <kbase name="rules" packages="rules">
            <ksession name="ksession-rules"/>
        </kbase>
        <kbase name="dtables" packages="dtables">
            <ksession name="ksession-dtables"/>
        </kbase>
    </kmodule>
     ```
    > * kbase name: 名字唯一标示
    > * packages: 资源文件所在的目录
    > * ksession name: 唯一标识
