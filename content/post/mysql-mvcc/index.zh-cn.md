---
title: MySQL >> MVCC
author: Jiaqi Gao
description: 
date: 2023-06-01T22:59:43+08:00
lastmod: 2023-06-01T22:59:43+08:00
slug: mysql-mvcc
image: 
math: false
license: 
hidden: false
comments: true
draft: false
categories:
  - DataSource
tags:
  - MySQL
  - DataSource
series:
aliases:
---
## 简介

MVCC全称Multi-Version Concurrency Control，即**多版本并发控制**。MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。

**多版本并发控制**：指的是一种高并发技术。最早的数据库系统，只有读读之间可以并发，读写，写读，写写都要阻塞。引入多版本之后，**只有写写之间相互阻塞，其他三种操作都可以并行**，这样大幅度提高了InnoDB的并发度。在内部实现中，InnoDB是基于`undo log`实现的，通过`undo log`可以找回数据的历史版本。找回的数据历史版本可以提供给用户读（按照隔离级别的定义，有些读请求只能看到比较老的数据版本），也可以在回滚的时候覆盖数据页上的数据。在InnoDB内部中，会记录一个全局的活跃读写事务数组，其主要用来判断事务的可见性。

> MVCC是一种多版本并发控制机制。

## 前置知识

#### InnoDB的当前读和快照读

* **当前读**：读取的记录是最新版本，读取时要保证其他事务不能修改当前记录，会对读取的记录进行加锁。
> 当前读主要包含以下场景
> * 共享锁：`select ... lock in share mode`
> * 排他锁：`select for update`，`insert`，`update`，`delete`

* **快照读**：**不加锁**的`select`操作属于快照读，即不加锁的非阻塞读。快照读基于多版本并发控制实现，即MVCC，正因为如此，可能导致快照读读取到的数据不是最新版本。
> 快照读的前提是事务的隔离级别不是串行级别，串行级别下快照读会退化成当前读。

MVCC就是为了实现读-写冲突不加锁，而这个读指的就是**快照读**，而非当前读，当前读实际上是一种加锁的操作，是悲观锁的实现。

#### 数据库的并发场景

* `读-读`：不存在线程安全问题，不需要并发控制。
* `读-写`：存在线程安全问题，可能会造成事务隔离性问题，可能遇到脏读，幻读，不可重复读
* `写-写`：存在线程安全问题，可能会存在更新丢失问题，如第一类更新丢失，第二类更新丢失
> * 第一类更新丢失：事务A撤销时，把已经提交的事务B的更新数据覆盖。
> * 第二类更新丢失：事务A覆盖事务B已经提交的数据，造成事务B所做的操作丢失。

## 实现原理

MVCC主要依赖记录中的**三个隐式字段**、**undo log**、**Read View**来实现的。

### 隐式字段

每行记录除了我们自定义的字段外，还有数据库隐式定义的`DB_TRX_ID`，`DB_ROLL_PTR`，`DB_ROW_ID`等字段
* `DB_TRX_ID`：`6byte`， 最后修改（插入/更新）的事务ID
* `DB_ROLL_PTR`：`7byte`，回滚指针，配合`undo log`指向上一个版本的记录（存储于`rollback segment`）
* `DB_ROW_ID`：`6byte`，隐含的自增ID（隐藏主键），如果数据表没有主键，InnoDB会自动以`DB_ROW_ID`产生一个聚簇索引

| Column1 | Column2 | DB_TRX_ID | DB_ROW_ID | DB_ROLL_PTR |
|:------- |:-------:|:---------:|:---------:|:-----------:|
| value1  | value2  |     1     |     1     | 0x12446462  |



### undo日志

* `insert undo log`：事务在`insert`新记录时产生的`undo log`，只在事务回滚时需要，并且在事务提交后可以被立即丢弃。
* `update undo log`：事务在进行`update`或`delete`时产生的`undo log`，不仅在事务回滚时需要，在快照读时也需要，所以不能随便删除，只有在快速读或事务回滚不涉及该日志时，对应的日志才会被`purge`线程统一清除。
> `purge`
> * 为了实现InnoDB的MVCC机制，更新或者删除操作都只是设置一下老记录的deleted_bit，并不真正将过时的记录删除。
> * 为了节省磁盘空间，InnoDB有专门的`purge`线程来清理`deleted_bit`为`true`的记录。为了不影响MVCC的正常工作，`purge`线程自己也维护了一个`read view`（这个`read view`相当于系统中最老活跃事务的`read view`），如果某个记录的`deleted_bit`为`true`，并且`DB_TRX_ID`相对于`purge`线程的`read view`可见，那么这条记录一定是可以被安全清除的。

MVCC依赖于`update undo log`，`undo log`实际上就是存在`rollback segment`中旧记录链。

### Read View

`Read View`是事务进行快照读操作时生产的读视图`Read View`，在该事务执行快照读的那一刻，会生成数据库系统当前的一个快照，记录并维护系统当前活跃事务的ID（当每个事务开启时，都会被分配一个ID, 这个ID是递增的，所以最新的事务，ID值越大）。

`Read View`主要是用来做可见性判断，`Read View`遵循一个可见性算法，主要是将**要被修改的数据**的最新记录中的`DB_TRX_ID`（即当前事务ID）取出来，与系统当前其他活跃事务的ID去对比（由Read View维护），如果`DB_TRX_ID`跟`Read View`的属性做了某些比较，不符合可见性，那就通过`DB_ROLL_PTR`回滚指针去取出`undo Log`中的`DB_TRX_ID`再比较，即遍历链表的`DB_TRX_ID`（从链首到链尾，即从最近的一次修改查起），直到找到满足特定条件的`DB_TRX_ID`，那么这个`DB_TRX_ID`所在的旧记录就是当前事务能看见的最新**老版本**。