---
title: 安装Nexus OSS 3.x（安装为Windows服务）
date: 2017-08-26 20:02:00
update: 2017-08-26 20:02:00
tags: [Nexus,Maven,工具,Windows]
categories: [开发环境]
---
## 下载安装包
下载地址：
https://www.sonatype.com/download-oss-sonatype

## 以Windows服务的方式运行 nexus oss

在Windows平台上运行Nexus Repository Manager Pro和Nexus Repository Manager OSS的启动脚本是 `bin/nexus.exe `。该脚本包括启动和停止服务的标准命令。它还包含命令 `install` 和 `uninstall` 以创建和删除服务的配置。您可以使用以下命令创建服务配置：

```shell
nexus.exe / install <服务名称（可选）>
```

默认情况下，新服务名为nexus。它在Windows控制台应用程序中(诸如Windows服务之类）可用于管理服务。你可以启动、停止或重新启动服务，并将其配置为作为操作系统启动的一部分启动。或者，您可以在命令行上管理服务：

```shell
nexus.exe / start <服务名称（可选）>
nexus.exe / stop <服务名称（可选）>
nexus.exe / uninstall <服务名称（可选）>
```
具有例如值的`<optional-service-name>`参数。 `nexus3`可用于创建不与在同一服务器上运行的`Nexus Repository Manager 2`建立的现有服务相冲突的服务。

> 在其他操作系统里的安装方式参看链接：https://help.sonatype.com/display/NXRM3/Installation
