---
title: 创建 PostgreSQL 用户和数据库
date: 2024-04-22 09:20:10
tags: [PostgreSQL, SQL]
---

# 简化 SQL 语句， 注意高版本的 PostgreSQL 需要使用 `GRANT` 命令

```sql

create role foo with login password 'yourpassword' valid until 'infinity';

-- 高版本需要使用这个命令
GRANT foo TO postgres;

create database bar with encoding='UTF8' owner=foo connection limit=-1;

REVOKE foo FROM postgres;

```

Reference: 
1. [ubuntu下postgreSQL安装配置](https://www.cnblogs.com/zhangpengshou/p/5464610.html)
2. [Backup&Restore](https://www.postgresql.org/docs/9.1/backup-dump.html#BACKUP-DUMP-RESTORE)