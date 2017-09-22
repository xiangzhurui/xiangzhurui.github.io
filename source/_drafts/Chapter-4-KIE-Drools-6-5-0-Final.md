---
title: 第四章 KIE (Drools 6.5.0.Final)
date: 2017-09-22 16:53:02
categories: [Drools]
tags: [Drools, KIE, 规则引擎, 文档, 翻译]
---

> KIE是Drools和jBPM的共享核心。它为构建，部署和利用资源提供了统一的方法论和编程模型。
> [原文](https://docs.jboss.org/drools/release/6.5.0.Final/drools-docs/html/ch04.html)

## 目录
4.1. 概览
    4.1.1. 项目剖析
    4.1.2. 生命周期
4.2. 构建，部署，使用和运行
    4.2.1. 介绍
    4.2.2. 建造
    4.2.3. 部署
    4.2.4. 运行
    4.2.5. 安装和部署作弊表
    4.2.6. 构建，部署和使用示例
4.3. 安全
    4.3.1. 安全管理

## 4.1 概览
### 4.4.1 项目剖析
研究Drools和jBPM的集成知识解决方案的过程只是使用“droolsjbpm”组名称。这个名字渗透了GitHub帐户和Maven POM。随着范围的扩大和新项目的推出，KIE被认为是“知识是一切”的首字母缩略词，作为新的组名。KIE名称也用于系统的共享方面; 如统一构建，部署和利用。

KIE目前由以下子项目组成：

图4.1. KIE解剖学

![](./第四章-KIE-Drools-6-5-0-Final/kie.png)

OptaPlanner是一种本地搜索和优化工具，已经从Drools Planner中脱颖而出，现在是Drools和jBPM的顶级项目。这是Optaplanner的一个自然演变，同时拥有强大的Drools集成，长期以来一直独立于Drools。

来自Polymita的收购以及其他功能来自强大的Dashboard Builder，它提供了强大的报告功能。 Dashboard Builder当前是一个临时名称，在6.0版本之后，将会选择一个新的名称。 Dashboard Builder完全独立于Drools和jBPM，并将被JBoss许多项目所使用，希望在JBoss之外:)

UberFire是新的基础工作台项目，从头开始重写。 UberFire提供类Eclipse的工作台功能，包括插件和面板。该项目独立于Drools和jBPM，任何人都可以将其用作构建灵活强大的工作台的基础。 UberFire将用于JBoss中的控制台和工作台开发。

确定Guvnor品牌从预期的角色泄漏太多;如创作隐喻，如决策表，被视为Guvnor组件而不是Drools组件。 Guvnor的5.x中使用的整体项目结构没有帮助。在6.0中，Guvnor的重点已经缩小到封装了一组UberFire插件，为构建基于Web的IDE提供了基础。例如用于构建和部署Maven集成，通过收件箱管理Maven存储库和活动通知。 Drools和jBPM使用Uberfire作为基础构建工作台分发，并包括一组插件，如Guvnor，以及自己的插件，如决策表，指导编辑，BPMN2设计人员和人事任务。 Drools工作台称为Drools-WB。 KIE-WB是将所有Guvnor，Drools和jBPM插件相结合的uber工作台。 jBPM-WB由于不存在，被KIE-WB冗余。
