# 索引与性能

## 0x01 索引基础

### 什么是索引？

> 索引是数据库中用于加速查询的数据结构。类似于书籍的目录，索引可以帮助数据库快速定位数据。

### 索引的优点和缺点

**优点：**
- 加速 SELECT 查询
- 加速 WHERE 条件过滤
- 加速 JOIN 操作
- 加速 ORDER BY 排序

**缺点：**
- 占用存储空间
- 降低 INSERT、UPDATE、DELETE 速度
- 需要维护成本

## 0x02 索引类型

### B-Tree 索引（默认）

最常用的索引类型，适用于等值查询和范围查询：

```sql
-- 创建 B-Tree 索引
CREATE INDEX idx_username ON users(username);

-- 创建唯一索引
CREATE UNIQUE INDEX idx_email ON users(email);

-- 创建复合索引
CREATE INDEX idx_name_email ON users(username, email);
```

### Hash 索引

适用于等值查询，不支持范围查询：

```sql
-- MySQL: Memory 引擎支持 Hash 索引
CREATE TABLE cache (
    id INT PRIMARY KEY,
    data TEXT
) ENGINE=MEMORY;

-- PostgreSQL: 创建 Hash 索引
CREATE INDEX idx_hash_username ON users USING HASH (username);
```

### 全文索引

适用于文本搜索：

```sql
-- MySQL 全文索引
CREATE FULLTEXT INDEX idx_fulltext_content ON articles(content);

-- 使用全文搜索
SELECT * FROM articles WHERE MATCH(content) AGAINST('database');

-- PostgreSQL 全文索引
CREATE INDEX idx_fts_content ON articles USING GIN (to_tsvector('english', content));

-- 使用全文搜索
SELECT * FROM articles WHERE to_tsvector('english', content) @@ to_tsquery('database');
```

### 空间索引

适用于地理空间数据：

```sql
-- MySQL 空间索引
CREATE SPATIAL INDEX idx_location ON places(location);

-- PostgreSQL 空间索引
CREATE INDEX idx_location ON places USING GIST (location);
```

## 0x03 索引操作

### 创建索引

```sql
-- 基本索引
CREATE INDEX idx_column ON table_name(column);

-- 唯一索引
CREATE UNIQUE INDEX idx_column ON table_name(column);

-- 复合索引
CREATE INDEX idx_columns ON table_name(column1, column2);

-- 部分索引（PostgreSQL）
CREATE INDEX idx_active_users ON users(username) WHERE status = 'active';

-- 表达式索引
CREATE INDEX idx_lower_username ON users(LOWER(username));
```

### 查看索引

```sql
-- MySQL
SHOW INDEX FROM table_name;

-- PostgreSQL
\di  -- 列出所有索引
\d table_name  -- 显示表的索引
SELECT * FROM pg_indexes WHERE tablename = 'table_name';
```

### 删除索引

```sql
-- 删除索引
DROP INDEX idx_column;

-- MySQL
DROP INDEX idx_column ON table_name;

-- PostgreSQL
DROP INDEX IF EXISTS idx_column;
```

## 0x04 查询优化

### EXPLAIN 分析

```sql
-- MySQL EXPLAIN
EXPLAIN SELECT * FROM users WHERE username = 'john';

-- 详细分析
EXPLAIN FORMAT=JSON SELECT * FROM users WHERE username = 'john';

-- PostgreSQL EXPLAIN
EXPLAIN SELECT * FROM users WHERE username = 'john';

-- 详细分析
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM users WHERE username = 'john';
```

### 索引使用原则

```sql
-- 1. 在 WHERE 条件列上创建索引
CREATE INDEX idx_status ON users(status);
SELECT * FROM users WHERE status = 'active';

-- 2. 在 JOIN 列上创建索引
CREATE INDEX idx_user_id ON orders(user_id);
SELECT * FROM users u JOIN orders o ON u.id = o.user_id;

-- 3. 在 ORDER BY 列上创建索引
CREATE INDEX idx_created_at ON users(created_at);
SELECT * FROM users ORDER BY created_at DESC;

-- 4. 复合索引的最左前缀原则
CREATE INDEX idx_name_email ON users(username, email);

-- 可以使用索引
SELECT * FROM users WHERE username = 'john';
SELECT * FROM users WHERE username = 'john' AND email = 'john@example.com';

-- 无法使用索引
SELECT * FROM users WHERE email = 'john@example.com';
```

### 查询优化技巧

```sql
-- 1. 避免 SELECT *
SELECT username, email FROM users;  -- 好
SELECT * FROM users;  -- 不好

-- 2. 使用 LIMIT 限制结果
SELECT * FROM users WHERE status = 'active' LIMIT 100;

-- 3. 避免在索引列上使用函数
-- 不好
SELECT * FROM users WHERE YEAR(created_at) = 2024;

-- 好
SELECT * FROM users WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01';

-- 4. 使用 EXISTS 代替 IN（大数据集）
-- 不好
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders);

-- 好
SELECT * FROM users u WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- 5. 避免使用 OR（可能导致索引失效）
-- 不好
SELECT * FROM users WHERE username = 'john' OR email = 'john@example.com';

-- 好
SELECT * FROM users WHERE username = 'john'
UNION ALL
SELECT * FROM users WHERE email = 'john@example.com';
```

## 0x05 性能监控

### MySQL 性能监控

```sql
-- 查看慢查询
SHOW VARIABLES LIKE 'slow_query%';
SHOW VARIABLES LIKE 'long_query_time';

-- 开启慢查询日志
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 2;  -- 2秒

-- 查看当前查询
SHOW PROCESSLIST;

-- 查看表状态
SHOW TABLE STATUS LIKE 'users';

-- 分析表
ANALYZE TABLE users;

-- 优化表
OPTIMIZE TABLE users;
```

### PostgreSQL 性能监控

```sql
-- 查看慢查询
SELECT * FROM pg_stat_activity WHERE state = 'active';

-- 查看表统计信息
SELECT * FROM pg_stat_user_tables WHERE relname = 'users';

-- 查看索引使用情况
SELECT * FROM pg_stat_user_indexes WHERE relname = 'users';

-- 分析表
ANALYZE users;

-- 重建索引
REINDEX TABLE users;
```

## 0x06 索引设计最佳实践

### 复合索引设计

```sql
-- 好的复合索引设计
-- 1. 等值查询列在前
CREATE INDEX idx_status_created ON orders(status, created_at);

-- 使用示例
SELECT * FROM orders WHERE status = 'pending' ORDER BY created_at DESC;

-- 2. 范围查询列在后
CREATE INDEX idx_user_date ON orders(user_id, created_at);

-- 使用示例
SELECT * FROM orders 
WHERE user_id = 123 
AND created_at >= '2024-01-01' 
AND created_at < '2025-01-01';
```

### 覆盖索引

```sql
-- 创建覆盖索引（包含查询所需的所有列）
CREATE INDEX idx_covering ON users(username, email, status);

-- 查询只需要访问索引，不需要回表
SELECT username, email, status FROM users WHERE username = 'john';
```

### 索引数量控制

```sql
-- 检查未使用的索引
-- MySQL
SELECT * FROM sys.schema_unused_indexes;

-- PostgreSQL
SELECT * FROM pg_stat_user_indexes 
WHERE idx_scan = 0  -- 从未使用过
AND schemaname = 'public';
```

## 0x07 分区表

### 范围分区

```sql
-- MySQL 范围分区
CREATE TABLE orders (
    id INT,
    order_date DATE,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (YEAR(order_date)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026)
);

-- PostgreSQL 范围分区
CREATE TABLE orders (
    id SERIAL,
    order_date DATE,
    amount DECIMAL(10,2)
) PARTITION BY RANGE (order_date);

CREATE TABLE orders_2023 PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2024-01-01');

CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

### 列表分区

```sql
-- MySQL 列表分区
CREATE TABLE users (
    id INT,
    country VARCHAR(2),
    username VARCHAR(50)
) PARTITION BY LIST COLUMNS(country) (
    PARTITION p_asia VALUES IN ('CN', 'JP', 'KR'),
    PARTITION p_europe VALUES IN ('UK', 'DE', 'FR'),
    PARTITION p_americas VALUES IN ('US', 'CA', 'MX')
);

-- PostgreSQL 列表分区
CREATE TABLE users (
    id SERIAL,
    country VARCHAR(2),
    username VARCHAR(50)
) PARTITION BY LIST (country);

CREATE TABLE users_asia PARTITION OF users
    FOR VALUES IN ('CN', 'JP', 'KR');
```

## 参考

- [MySQL Index](https://dev.mysql.com/doc/refman/8.0/en/optimization-indexes.html)
- [PostgreSQL Indexes](https://www.postgresql.org/docs/13/indexes.html)
- [MySQL EXPLAIN](https://dev.mysql.com/doc/refman/8.0/en/using-explain.html)
- [PostgreSQL EXPLAIN](https://www.postgresql.org/docs/13/using-explain.html)
- [MySQL Partitioning](https://dev.mysql.com/doc/refman/8.0/en/partitioning.html)
- [PostgreSQL Partitioning](https://www.postgresql.org/docs/13/ddl-partitioning.html)