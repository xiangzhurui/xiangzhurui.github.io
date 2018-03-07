---
title: 常见应用场景下的 Git 命令
tags:
    - git
    - 版本控制
---

1. 从 版本控制 中移除eclipse 文件
    ```bash
    git rm --cached --force .project
    git rm --cached --force -r /.settings
    ```
3. git stash
    ```bash
    ```
4. 修改远程仓库地址
    ```bash
    #方法一：
    git remote set-url origin git@192.168.1.18:mStar/OTT-dual/K3S/supernova

    #方法二：
    git remote rm origin
    git remote add origin git@192.168.1.18:mStar/OTT-dual/K3S/supernova
    ```
5. 修改分支名称
    ```bash
    # 删除远程分支
    git push --delete origin devel
    # 解除本地分支与远程分支关联
    # 修改本地分支名
    git branch -m devel develop
    # 推送本地分支到远端
    git push origin develop
    ```
6. 重置工作副本
    ```bash
    git reset --hard
    git clean -xdf
    ```
7. 比较两个分支
