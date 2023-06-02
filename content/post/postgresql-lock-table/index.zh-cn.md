---
title: PostgreSQL 锁表解决方案
author: Jiaqi Gao
description: 
date: 2023-06-03T00:09:29+08:00
lastmod: 2023-06-03T00:09:29+08:00
slug: postgresql-lock-table
image: 
math: false
license: 
hidden: false
comments: true
draft: false
categories: 
  - Database
tags:
  - Database
  - PostgreSQL
series:
aliases:
---

## 根据表名查询`pid`列表

```SQL
select locks.pid
from pg_locks locks
inner join pg_class class
on locks.relation = class.oid
where class.relname = 'table_name';
```

## 使用终止函数终止`pid`

```SQL
select pg_terminate_backend(pid);
```