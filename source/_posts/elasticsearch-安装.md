---
title: Elasticsearch 安装
tags:
  - elasticsearch
  - 工具
  - 文档
date: 2017-08-12 12:00:30
update: 2017-08-12 12:00:30
categories: [开发环境]
---


> Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。 它能从项目一开始就赋予你的数据以搜索、分析和探索的能力，这是通常没有预料到的。 它存在还因为原始数据如果只是躺在磁盘里面根本就毫无用处。

## Elasticsearch 文档

* [Elasticsearch 网站](https://www.elastic.co/)
* [Elasticsearch: 权威指南](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/index.html)
* [Elasticsearch Clients - Java REST Client [5.5]](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)

## 安装
> 本文使用 Windows 10 下操作，elasticsearch 需要 jdk 1.8

一下链接有详细安装步骤，简单而言就是下载程序包，解压后运行脚本,`Elasticsearch` 运行 `.\bin\elasticsearch.bat`, `Kibana` 是运行`.\bin\kibana.bat`
* [安装 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html#install-elasticsearch)
* [安装 Kibana](https://www.elastic.co/guide/en/kibana/current/install.html)
* [NSSM}(https://nssm.cc/download)

### 快速启动开发环境
    * 如果你想把 Elasticsearch 作为一个守护进程在后台运行，那么可以在后面添加参数 -d 。
    * 执行 `./elasticsearch` 启动 `elasticsearch` 服务。
    * 执行 `./kibana`  启动 `kibana` 服务。
### kibana
以下地址可访问，elasticsearch 需要 jdk 1.8
http://localhost:5601

> 现在已经可以使用了，其他安装配置事项待续。。。
