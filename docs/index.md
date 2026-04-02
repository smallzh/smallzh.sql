# smallzh's SQL 知识库

欢迎来到 SQL 核心知识库！本知识库涵盖 SQL 的基础到常用知识点，主要面向 MySQL 8+ 和 PostgreSQL 13+。

## 基础

- [SQL基础概念](basics/concepts.md) - 什么是SQL，数据库基本概念，SQL语言分类
- [数据类型](basics/data-types.md) - 数值、字符串、日期时间、二进制等数据类型
- [基本操作](basics/basic-operations.md) - DDL（创建、修改、删除表）和DML（增删改查）
- [查询基础](basics/query-basics.md) - SELECT、WHERE、ORDER BY、LIMIT等基础查询

## 常用

- [高级查询](common/advanced-queries.md) - JOIN连接、子查询、聚合函数、窗口函数
- [常用函数](common/functions.md) - 字符串、数值、日期时间、条件、JSON函数
- [索引与性能](common/indexes-performance.md) - 索引类型、创建、查询优化、性能监控
- [事务与并发](common/transactions-concurrency.md) - 事务ACID、隔离级别、锁机制、MVCC

## 使用说明

### 本地运行

```shell
# 安装依赖
uv sync

# 启动本地开发服务器
uv run mkdocs serve
```

### 构建静态站点

```shell
uv run mkdocs build
```

## 环境要求

- Python >= 3.13
- MkDocs >= 1.6.1
- mkdocs-smzhbook-theme >= 1.1

## 参考

- [MySQL 8.0 官方文档](https://dev.mysql.com/doc/refman/8.0/en/)
- [PostgreSQL 13 官方文档](https://www.postgresql.org/docs/13/)
- [SQL 教程 - W3Schools](https://www.w3schools.com/sql/)