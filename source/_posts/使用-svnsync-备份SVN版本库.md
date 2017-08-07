---
title: 使用 svnsync 备份SVN版本库
date: 2017-08-07 17:41:34
tags: [svnsync, SVN, Subversion, 版本控制]
---

> 本文操作环境为 Windows 10 与 PowerShell


```shell
# 在 PowerShell 或 bash 里运行

# 1. 建立同步关联
svnsync init "file:///D:/WorkDir/Subversion/Repositories/repo-sample" http://192.168.1.55:443/remote-repo

#
svnsync --allow-non-empty init  "file:///D:/WorkDir/Subversion/Repositories/repo-sample"  http://192.168.1.55:443/remote-repo

# 开始同步
svnsync sync "file:///D:/WorkDir/Subversion/Repositories/repo-sample"
```

## 错误解决
1. 错误提示1：`Repository has not been enabled to accept revision propchanges`
hooks 目录下新建 `pre-revprop-change.bat`

2. 错误提示2： `svnsync: E000022: Couldn't get lock on destination repos after 10 attempts`
    
    ```shell
    PS C:\Windows\system32> svnsync sync "file:///D:/WorkDir/Subversion/Repositories/repo-sample"
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    Failed to get lock on destination repos, currently held by 'DESKTOP-xxxxxxx:f24349b9-b65b-464e-8339-1e4a25d3906a'
    \`svnsync: E000022: Couldn\'t get lock on destination repos after 10 attempts \`
    ```

    这个时候可能属性被锁了，删掉属性：

    ```shell
    svn propdel svn:sync-lock --revprop -r0 "file:///D:/WorkDir/Subversion/Repositories/repo-sample"
    ```

## SVN 库的备份与迁移
    ```shell
    ## 备份svn库
    svnadmin dump D:\Subversion\Repositories\repo-src > D:\backups\repo.bak
    ## 恢复svn库
    svnadmin load D:\backups\single-repository --parent-dir aaa < D:\backups\repo.bak
    ```

## 参考资料：
https://www.visualsvn.com/support/svnbook/ref/svnsync/c/init/
