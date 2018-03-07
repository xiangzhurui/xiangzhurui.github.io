---
title: Mac下常见场景的命令
tags:
---

1. 递归删除.git目录
```bash
find . -name ".git" -type d -exec rm -rf {} \;
```
2. 递归删除后缀为.iml文件
```bash
find . -name "*.iml" -type f -exec rm -rf {} \;
```
