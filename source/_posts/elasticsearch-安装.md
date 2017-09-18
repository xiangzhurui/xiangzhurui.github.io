---
title: Elasticsearch 安装
tags:
  - Elasticsearch
  - 工具
  - 文档
date: 2017-08-12 12:00:30
update: 2017-09-18 12:30:30
categories: [开发环境]
---


> Elasticsearch 是一个分布式、可扩展、实时的搜索与数据分析引擎。 它能从项目一开始就赋予你的数据以搜索、分析和探索的能力，这是通常没有预料到的。 它存在还因为原始数据如果只是躺在磁盘里面根本就毫无用处。

## Elasticsearch 文档

* [Elasticsearch 网站](https://www.elastic.co/)
* [Elasticsearch: 权威指南](https://elasticsearch.cn/book/elasticsearch_definitive_guide_2.x/index.html)
* [Elasticsearch Clients - Java REST Client [5.5]](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/current/index.html)

## 简述
> Elasticsearch 需要 jdk 1.8

以下链接有详细安装步骤，简单而言就是下载程序包，解压后运行脚本,`Elasticsearch` 运行 `.\bin\elasticsearch.bat`, `Kibana` 是运行`.\bin\kibana.bat`
* [安装 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html#install-elasticsearch)
* [安装 Kibana](https://www.elastic.co/guide/en/kibana/current/install.html)
* [NSSM}(https://nssm.cc/download)

## Elasticsearch 安装步骤

1. 下载并解压 Elasticsearch
    >注意 :Elasticsearch也可以使用apt或yum从我们的软件包仓库安装，或者使用MSI安装程序包安装在Windows上。请参阅指南中的存储库。
2. 运行 `bin/elasticsearch`（或在Windows上运行 `bin\elasticsearch.bat`）
3. 运行 `curl http://localhost:9200/` 或使用 PowerShell 运行 `Invoke-RestMethod http://localhost:9200`
4. 查阅[入门指南](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started.html)

## Kibana 安装步骤
1. 下载并解压缩Kibana
    > 还可以使用apt或yum从我们的软件包仓库安装Kibana。请参阅[存储库指南](https://www.elastic.co/guide/en/kibana/current/install.html)。
2. 在文本编辑器中打开 `config/kibana.yml` ,设置 `elasticsearch.url` 只想你的 Elasticsearch 实例
3. 运行 `bin/kibana`（或在Windows上运行 `bin\kibana.bat` ）
4. 使用浏览器访问 http://localhost:5601
5. 查阅[入门指南](https://www.elastic.co/guide/en/kibana/current/getting-started.html) 。


### 快速启动开发环境
1. 如果你想把 Elasticsearch 作为一个守护进程在后台运行，那么可以在后面添加参数 -d 。
2. 执行 `./elasticsearch` 启动 `elasticsearch` 服务。
3. 执行 `./kibana`  启动 `kibana` 服务。
4. 访问 kibana ： http://localhost:5601
