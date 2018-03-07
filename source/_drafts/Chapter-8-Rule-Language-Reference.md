---
title: 第8章、规则语言参考
tags: [Drools, Drools 6.5.0.Final,规则引擎,语言规范]
categories: [Drools]
---
## 目录：
8.1. Overview
8.1.1. A rule file
8.1.2. What makes a rule
8.2. Keywords
8.3. Comments
8.3.1. Single line comment
8.3.2. Multi-line comment
8.4. Error Messages
8.4.1. Message format
8.4.2. Error Messages Description
8.4.3. Other Messages
8.5. Package
8.5.1. import
8.5.2. global
8.6. Function
8.7. Type Declaration
8.7.1. Declaring New Types
8.7.2. Declaring Metadata
8.7.3. Declaring Metadata for Existing Types
8.7.4. Parametrized constructors for declared types
8.7.5. Non Typesafe Classes
8.7.6. Accessing Declared Types from the Application Code
8.7.7. Type Declaration 'extends'
8.7.8. Traits
8.8. Rule
8.8.1. Rule Attributes
8.8.2. Timers and Calendars
8.8.3. Left Hand Side (when) syntax
8.8.4. The Right Hand Side (then)
8.8.5. Conditional named consequences
8.8.6. A Note on Auto-boxing and Primitive Types
8.9. Query
8.10. Domain Specific Languages
8.10.1. When to Use a DSL
8.10.2. DSL Basics
8.10.3. Adding Constraints to Facts
8.10.4. Developing a DSL
8.10.5. DSL and DSLR Reference

## 8.1 概览
Drools具有“本机”规则语言。这种格式在标点符号方面非常轻巧，并且通过“扩展器”支持自然和特定于域的语言，允许语言转变为您的问题域。本章大部分与本地规则格式一致。用于呈现语法的图表被称为“铁路”图，它们基本上是语言术语的流程图。技术上非常热衷于DRL.g ，它是规则语言的Antlr3语法。如果您使用Rule Workbench，则可以通过内容帮助为您完成许多规则结构，例如，键入“ru”并按ctrl + space，它将为您构建规则结构。

### 8.1.1 规则文件
规则文件通常是具有.drl扩展名的文件。在一个DRL文件中，您可以拥有多个规则，查询和函数，以及您的规则和查询分配和使用的一些资源声明，如导入，全局变量和属性。但是，您也可以将规则跨越多个规则文件（在这种情况下，扩展名.rule是建议的，但不是必需的） - 跨文件扩展规则可以帮助管理大量规则。一个DRL文件只是一个文本文件。
规则文件的总体结构如下：
