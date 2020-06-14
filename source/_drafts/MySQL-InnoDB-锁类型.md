---
title: MySQL InnoDB 锁类型
tags:
- MySQL
- InnoDB
- Lock
---

> InnoDB 使用的锁定类型 

* 共享锁和排它锁 （Shared and Exclusive Locks)
* 意向锁 (Intention Locks)
* 记录所（行锁、Record Locks）
* 间隙锁 (Gap Locks)
* 下一键锁 (Next-Key Locks)
* 插入意向锁 (Insert Intention Locks)
* 自增锁 (AUTO-INC Locks)
* 空间索引条件锁 (Predicate Locks for Spatial Indexs)

## 共享锁和排它锁 （Shared and Exclusive Locks)

InnoDB 实现了标准的行级锁，包含有两种类型的锁： 共享（S）锁和排他（X）锁。
* 共享锁允许持有该锁的事务读取一行
* 排它锁允许你持有该锁的事务更新或删除一行

如果事务 T1 是有一个在行 r 上的共享锁 ，那么来自某些不同事务T2的对行r的锁请求将按以下方式处理：
* 来自T2的S锁请求将可以立即获取到。结果上，T1 和 T2 都持有在行 r 上的 S 锁。
* 来自T2的X锁请求将无法立即获取到。

如果事务T1在行 r 上拥有排它锁（X），则某个不同事务T2的任意类型锁请求都不能立即获取到。然后，事务T2 必须等待到T1释放对行 r 的锁，才能获取到锁。

## 意向锁 （Intention Locks)

InnoDB 支持多重粒度的锁，允许行锁和表锁并存。例如，

## 自增锁 （AUTO-INC Locks)

自增锁是一个特殊的表级锁

