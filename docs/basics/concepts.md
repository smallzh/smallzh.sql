# SQL基础概念

## 0x01 什么是SQL？

> SQL (Structured Query Language) 是一种用于管理关系型数据库的标准语言。它用于创建、查询、更新和管理数据库中的数据。

SQL 是数据库的通用语言，支持 MySQL、PostgreSQL、Oracle、SQL Server 等多种数据库系统。

## 0x02 数据库基本概念

### 关系型数据库

关系型数据库基于关系模型，使用表（Table）来存储数据。每个表由行（Row）和列（Column）组成。

```sql
-- 示例：创建一个简单的用户表
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(50),
    email VARCHAR(100),
    created_at TIMESTAMP
);
```

### 核心术语

| 术语 | 说明 | 示例 |
|------|------|------|
| 数据库（Database） | 数据的集合 | `mydb` |
| 表（Table） | 数据的结构化存储 | `users` |
| 行（Row）/记录（Record） | 表中的一条数据 | 一个用户信息 |
| 列（Column）/字段（Field） | 数据的属性 | `username`, `email` |
| 主键（Primary Key） | 唯一标识每行的字段 | `id` |
| 外键（Foreign Key） | 关联其他表的字段 | `user_id` |

## 0x03 SQL语言分类

SQL 语言根据功能分为以下几类：

### DDL（Data Definition Language）- 数据定义语言

用于定义数据库结构：

```sql
-- 创建表
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL(10,2)
);

-- 修改表结构
ALTER TABLE products ADD COLUMN description TEXT;

-- 删除表
DROP TABLE products;
```

### DML（Data Manipulation Language）- 数据操作语言

用于操作数据：

```sql
-- 插入数据
INSERT INTO products (id, name, price) VALUES (1, 'Laptop', 999.99);

-- 更新数据
UPDATE products SET price = 899.99 WHERE id = 1;

-- 删除数据
DELETE FROM products WHERE id = 1;
```

### DQL（Data Query Language）- 数据查询语言

用于查询数据：

```sql
-- 查询所有数据
SELECT * FROM products;

-- 条件查询
SELECT name, price FROM products WHERE price < 1000;

-- 排序查询
SELECT * FROM products ORDER BY price DESC;
```

### DCL（Data Control Language）- 数据控制语言

用于控制访问权限：

```sql
-- 授予权限
GRANT SELECT, INSERT ON products TO user1;

-- 撤销权限
REVOKE INSERT ON products FROM user1;
```

## 0x04 SQL语句基本结构

SQL 语句通常由关键字、表名、列名和条件组成：

```sql
SELECT column1, column2    -- 要查询的列
FROM table_name            -- 要查询的表
WHERE condition            -- 查询条件
ORDER BY column1           -- 排序方式
LIMIT 10;                  -- 限制结果数量
```

## 0x05 注释

SQL 支持单行和多行注释：

```sql
-- 这是单行注释

/* 这是多行注释
   可以跨越多行 */

SELECT * FROM users; -- 行尾注释
```

## 0x06 数据库设计原则

### 范式（Normalization）

数据库设计遵循范式原则，以减少数据冗余：

- **第一范式（1NF）**：确保每列都是原子性的
- **第二范式（2NF）**：满足1NF，且非主键列完全依赖于主键
- **第三范式（3NF）**：满足2NF，且非主键列不依赖于其他非主键列

```sql
-- 示例：符合3NF的设计
CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE,
    total_amount DECIMAL(10,2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

## 参考

- [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)
- [PostgreSQL 13 Documentation](https://www.postgresql.org/docs/13/)
- [SQL Tutorial - W3Schools](https://www.w3schools.com/sql/)