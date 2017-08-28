---
title: Elasticsearch API 使用
date: 2017-08-21 09:34:00
tags: [Elasticsearch]
update:
---

### 对应于关系数据库术语

Index == Database
Types == Tables
Document == row
Field == column
Mapping == Schema

文档是Elasticsearch的数据单位

### API 格式
```
# Http api 格式
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
# 上述的<PATH> 的格式
<index>/<type>/[<id>]
```
