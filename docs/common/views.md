# 视图

## 0x01 视图简介

视图（View）是一个虚拟表，其内容由查询定义。视图不存储实际数据，而是存储一个查询定义，每次访问视图时执行该查询。

```sql
-- 视图的本质是一个保存的查询
-- 视图 = 存储的查询定义
```

## 0x02 创建视图

### 基本语法

```sql
-- MySQL / PostgreSQL
CREATE VIEW view_name AS
SELECT column1, column2, ...
FROM table_name
WHERE condition;
```

### 示例

```sql
-- 创建用户订单视图
CREATE VIEW user_orders_view AS
SELECT 
    u.id AS user_id,
    u.name AS user_name,
    o.order_id,
    o.total_amount,
    o.created_at
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE o.status = 'completed';

-- 创建销售统计视图
CREATE VIEW monthly_sales AS
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_sales
FROM orders
GROUP BY DATE_TRUNC('month', order_date);
```

## 0x03 查询视图

```sql
-- 查询视图与查询表完全相同
SELECT * FROM user_orders_view;

-- 带条件查询
SELECT * FROM user_orders_view 
WHERE user_id = 1;

-- 视图可以与其他表 JOIN
SELECT v.*, p.product_name
FROM user_orders_view v
JOIN order_items oi ON v.order_id = oi.order_id
JOIN products p ON oi.product_id = p.id;
```

## 0x04 查看视图信息

```sql
-- MySQL: 查看视图定义
SHOW CREATE VIEW view_name;

-- PostgreSQL: 查看视图定义
SELECT pg_get_viewdef('view_name', true);

-- 查看所有视图
-- MySQL
SELECT TABLE_NAME 
FROM INFORMATION_SCHEMA.VIEWS 
WHERE TABLE_SCHEMA = 'database_name';

-- PostgreSQL
SELECT tablename 
FROM pg_tables 
WHERE schemaname = 'public' 
AND tablename LIKE '%\_view' ESCAPE '\';
```

## 0x05 修改视图

### 创建或替换视图

```sql
-- MySQL 8.0+
CREATE OR REPLACE VIEW view_name AS
SELECT ...
```

```sql
-- PostgreSQL
CREATE OR REPLACE VIEW view_name AS
SELECT ...
```

### 使用 ALTER 修改

```sql
-- PostgreSQL 支持 ALTER VIEW
ALTER VIEW view_name 
ALTER COLUMN column_name TYPE new_type;
```

## 0x06 删除视图

```sql
-- 删除视图
DROP VIEW view_name;

-- 如果存在则删除（MySQL 8.0+ / PostgreSQL）
DROP VIEW IF EXISTS view_name;

-- 删除多个视图
DROP VIEW view1, view2, view3;
```

## 0x07 视图的分类

### 简单视图

基于单个表创建的视图，不包含聚合函数或分组。

```sql
CREATE VIEW simple_users AS
SELECT id, name, email
FROM users
WHERE status = 'active';
```

### 复杂视图

包含聚合函数、JOIN、分组等复杂查询的视图。

```sql
CREATE VIEW complex_sales AS
SELECT 
    u.name,
    COUNT(o.id) AS order_count,
    SUM(o.total_amount) AS total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.name;
```

### 可更新视图

可以通过视图更新底层数据（需要满足特定条件）。

```sql
-- 可更新视图的条件：
-- 1. 不包含聚合函数
-- 2. 不包含 DISTINCT
-- 3. 不包含 GROUP BY / HAVING
-- 4. 不包含 UNION
-- 5. 视图的每列对应底层表的列

-- 通过视图更新数据
CREATE VIEW updatable_user AS
SELECT id, name, email
FROM users
WHERE status = 'active';

UPDATE updatable_user 
SET email = 'new@email.com' 
WHERE id = 1;
```

## 0x08 物化视图

物化视图（Materialized View）存储实际数据，定期刷新。

### PostgreSQL 物化视图

```sql
-- 创建物化视图
CREATE MATERIALIZED VIEW monthly_sales_mv AS
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_sales
FROM orders
GROUP BY DATE_TRUNC('month', order_date);

-- 刷新物化视图
REFRESH MATERIALIZED VIEW monthly_sales_mv;

-- 增量刷新（如果使用了 CONCURRENTLY）
REFRESH MATERIALIZED VIEW CONCURRENTLY monthly_sales_mv;

-- 查看物化视图
SELECT * FROM monthly_sales_mv;
```

### MySQL 物化视图

MySQL 不原生支持物化视图，可通过触发器或定时任务模拟。

## 0x09 视图的优缺点

### 优点

```sql
-- 1. 简化复杂查询
-- 多次使用的复杂查询可以保存为视图
CREATE VIEW complex_report AS
SELECT ... FROM multiple_tables WHERE ...;

-- 2. 数据安全
-- 只暴露需要的列，隐藏敏感数据
CREATE VIEW safe_user_info AS
SELECT id, name, email
FROM users;  -- 不包含 password, phone 等敏感字段

-- 3. 数据抽象
-- 改变底层表结构不影响应用，只需修改视图定义

-- 4. 代码复用
-- 多个模块使用相同查询逻辑，通过视图统一管理
```

### 缺点

```sql
-- 1. 性能问题
-- 每次查询视图都会执行底层查询，大数据量时可能慢

-- 2. 更新限制
-- 并非所有视图都可更新

-- 3. 依赖关系
-- 底层表结构变更可能影响视图，需要同步维护
```

## 0x10 视图的最佳实践

```sql
-- 1. 为视图命名添加约定前缀
CREATE VIEW v_user_active AS ...;
CREATE VIEW mv_monthly_sales AS ...;  -- 物化视图

-- 2. 添加注释说明视图用途
COMMENT ON VIEW v_user_active IS '活跃用户列表，用于前端展示';

-- 3. 避免嵌套视图（视图依赖视图）
-- 嵌套视图会导致性能问题和维护困难

-- 4. 定期检查视图定义是否仍然需要
-- 清理不再使用的视图

-- 5. 物化视图用于报表和统计
-- 适合变化不频繁但查询频繁的场景
```

## 参考

- [MySQL Views](https://dev.mysql.com/doc/refman/8.0/en/views.html)
- [PostgreSQL Views](https://www.postgresql.org/docs/13/sql-createview.html)
- [PostgreSQL Materialized Views](https://www.postgresql.org/docs/13/sql-creatematerializedview.html)