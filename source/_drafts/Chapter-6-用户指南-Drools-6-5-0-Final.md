---
title: 第六章 用户指南 (Drools 6.5.0.Final)
date: 2017-09-22 17:06:46
categories: [Drools]
tags: [Drools, 规则引擎]
---
* 目录

6.1. 基础
6.1.1. Stateless Knowledge Session
6.1.2. Stateful Knowledge Session
6.1.3. Methods versus Rules
6.1.4. Cross Products
6.2. Execution Control
6.2.1. Agenda
6.2.2. Rule Matches and Conflict Sets.
6.3. Inference
6.3.1. Bus Pass Example
6.4. Truth Maintenance with Logical Objects
6.4.1. Overview
6.5. Decision Tables in Spreadsheets
6.5.1. When to Use Decision Tables
6.5.2. Overview
6.5.3. How Decision Tables Work
6.5.4. Spreadsheet Syntax
6.5.5. Creating and integrating Spreadsheet based Decision Tables
6.5.6. Managing Business Rules in Decision Tables
6.5.7. Rule Templates
6.6. Logging
---
## 6.1. 基础

### 6.1.1. 无状态知识回话

那么我们从哪里开始呢？ 在诸如Drools这样的规则引擎中有如此多的用例和许多功能，它变得令人难以置信。 勇敢者不会恐惧，复杂性是分层的，你可以用简单的例子来更容易的学习。

无状态会话，不使用推理，形成最简单的用例。无状态会话可以被称为函数传递一些数据，然后再接收一些结果。无状态会话的一些常见用例是但不限于：
- 验证
  - 这个人有资格获得抵押贷款吗？
- 计算
  - 计算抵押保费。
- 路由和过滤
  - 将传入的信息（如电子邮件）过滤后放入到文件夹中。
  - 将传入的消息发送到目的地。

所以让我们从使用驾驶执照应用程序的一个非常简单的例子开始吧。

```java
public class Applicant {
    private String name;
    private int age;
    private boolean valid;
    // getter and setter methods here
}
```
现在我们有了我们的数据模型，我们可以写下我们的第一条规则 我们假设应用程序使用规则来拒绝无效的申请。由于这是一个简单的验证用例，我们将添加一条规则来取消18岁以下申请人的资格。
```java
package com.company.license

rule "Is of valid age"
when
    $a : Applicant( age < 18 )
then
    $a.setValid( false );
end
```

### 6.1.2. 有状态会话
### 6.1.3. 方法与规则
### 6.1.4. 交叉产品
之前提到“交叉产品(cross product)”一词，这是联结的结果。想象一下，来自火灾报警示例的数据与以下规则结合使用，其中没有字段约束：
```java
rule "Show Sprinklers" when
    $room : Room()
    $sprinkler : Sprinkler()
then
    System.out.println( "room:" + $room.getName() +
                        " sprinkler:" + $sprinkler.getRoom().getName() );
end
```

在SQL术语中，这将像在`select * from Room, Sprinkler` , Room表中的每一行将与Sprinkler表中的每一行相连接，从而产生以下输出：

```
room:office sprinkler:office
room:office sprinkler:kitchen
room:office sprinkler:livingroom
room:office sprinkler:bedroom
room:kitchen sprinkler:office
room:kitchen sprinkler:kitchen
room:kitchen sprinkler:livingroom
room:kitchen sprinkler:bedroom
room:livingroom sprinkler:office
room:livingroom sprinkler:kitchen
room:livingroom sprinkler:livingroom
room:livingroom sprinkler:bedroom
room:bedroom sprinkler:office
room:bedroom sprinkler:kitchen
room:bedroom sprinkler:livingroom
room:bedroom sprinkler:bedroom
```



## 6.2. 执行控制
### 6.2.1. 议程
### 6.2.2. 规则匹配和冲突集合

## 6.3. 推理
### 6.3.1. Bus 通过示例

## 6.4. 用逻辑对象维护真理
### 6.4.1. 概览


## 6.5. 电子表格中的决策表
### 6.5.1. 何时使用决策表
### 6.5.2. 概览
### 6.5.3. 决策表如何运作
### 6.5.4. 电子表格语法
### 6.5.5. 创建和集成基于电子表格的决策表
### 6.5.6. 在决策表中管理业务规则
### 6.5.7. 规则模板

## 6.6. 记录
