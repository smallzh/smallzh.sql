# 查询基础

## 0x01 SELECT 基础

### 基本查询

```sql
-- 查询所有列
SELECT * FROM users;

-- 查询特定列
SELECT username, email FROM users;

-- 查询表达式
SELECT username, age, age * 12 AS age_in_months FROM users;
```

### 别名

```sql
-- 列别名
SELECT username AS user_name, email AS user_email FROM users;

-- 表别名
SELECT u.username, u.email FROM users AS u;

-- 简写形式（省略 AS）
SELECT username user_name, email user_email FROM users u;
```

## 0x02 WHERE 条件过滤

### 比较运算符

```sql
-- 等于
SELECT * FROM users WHERE age = 25;

-- 不等于
SELECT * FROM users WHERE status != 'inactive';
SELECT * FROM users WHERE status <> 'inactive';  -- 另一种写法

-- 大于/小于
SELECT * FROM users WHERE age > 18;
SELECT * FROM users WHERE age >= 18;
SELECT * FROM users WHERE age < 65;
SELECT * FROM users WHERE age <= 65;
```

### 逻辑运算符

```sql
-- AND: 两个条件都满足
SELECT * FROM users WHERE age >= 18 AND age <= 65;

-- OR: 满足任一条件
SELECT * FROM users WHERE age < 18 OR age > 65;

-- NOT: 条件取反
SELECT * FROM users WHERE NOT status = 'inactive';

-- 组合使用
SELECT * FROM users WHERE (age >= 18 AND age <= 65) AND status = 'active';
```

### 范围查询

```sql
-- BETWEEN: 范围查询（包含边界）
SELECT * FROM users WHERE age BETWEEN 18 AND 65;

-- 等价于
SELECT * FROM users WHERE age >= 18 AND age <= 65;

-- NOT BETWEEN: 范围外查询
SELECT * FROM users WHERE age NOT BETWEEN 18 AND 65;
```

### 列表查询

```sql
-- IN: 在列表中
SELECT * FROM users WHERE country IN ('CN', 'US', 'JP');

-- 等价于
SELECT * FROM users WHERE country = 'CN' OR country = 'US' OR country = 'JP';

-- NOT IN: 不在列表中
SELECT * FROM users WHERE country NOT IN ('CN', 'US');

-- 子查询
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders);
```

### 模式匹配

```sql
-- LIKE: 模式匹配
-- % 匹配任意字符（0个或多个）
-- _ 匹配单个字符

-- 以 'john' 开头
SELECT * FROM users WHERE username LIKE 'john%';

-- 以 'gmail.com' 结尾
SELECT * FROM users WHERE email LIKE '%@gmail.com';

-- 包含 'admin'
SELECT * FROM users WHERE username LIKE '%admin%';

-- 第二个字符是 'a'
SELECT * FROM users WHERE username LIKE '_a%';

-- NOT LIKE: 不匹配模式
SELECT * FROM users WHERE username NOT LIKE 'test%';
```

### NULL 值处理

```sql
-- 检查 NULL 值
SELECT * FROM users WHERE phone IS NULL;

-- 检查非 NULL 值
SELECT * FROM users WHERE phone IS NOT NULL;

-- 注意：NULL 不能用 = 或 != 比较
-- 错误写法
SELECT * FROM users WHERE phone = NULL;  -- 不会返回任何结果

-- 正确写法
SELECT * FROM users WHERE phone IS NULL;
```

## 0x03 ORDER BY 排序

### 基本排序

```sql
-- 升序排序（默认）
SELECT * FROM users ORDER BY age ASC;
SELECT * FROM users ORDER BY age;  -- ASC 可省略

-- 降序排序
SELECT * FROM users ORDER BY age DESC;
```

### 多列排序

```sql
-- 先按 age 升序，再按 username 降序
SELECT * FROM users ORDER BY age ASC, username DESC;

-- 先按 status 分组，再按 created_at 降序
SELECT * FROM users ORDER BY status, created_at DESC;
```

### 表达式排序

```sql
-- 按表达式排序
SELECT * FROM users ORDER BY LENGTH(username);

-- 按别名排序
SELECT username, age * 12 AS age_in_months 
FROM users 
ORDER BY age_in_months DESC;

-- 按列位置排序（不推荐）
SELECT username, age FROM users ORDER BY 2;  -- 按第2列排序
```

## 0x04 LIMIT 分页

### 基本分页

```sql
-- MySQL/PostgreSQL
SELECT * FROM users ORDER BY id LIMIT 10;  -- 前10条

-- 跳过前5条，取10条
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 5;

-- MySQL 简写形式
SELECT * FROM users ORDER BY id LIMIT 5, 10;  -- 跳过5条，取10条
```

### 分页查询示例

```sql
-- 第1页（每页10条）
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 0;

-- 第2页
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 10;

-- 第3页
SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 20;

-- 通用分页公式
-- LIMIT page_size OFFSET (page_number - 1) * page_size
```

## 0x05 DISTINCT 去重

### 基本去重

```sql
-- 去除重复值
SELECT DISTINCT country FROM users;

-- 多列去重
SELECT DISTINCT country, city FROM users;
```

### 聚合去重

```sql
-- 统计不重复的国家数量
SELECT COUNT(DISTINCT country) FROM users;

-- MySQL 也支持
SELECT COUNT(DISTINCT country) AS country_count FROM users;
```

## 0x06 条件表达式

### CASE 表达式

```sql
-- 简单 CASE
SELECT 
    username,
    status,
    CASE status
        WHEN 'active' THEN '活跃'
        WHEN 'inactive' THEN '非活跃'
        ELSE '未知'
    END AS status_text
FROM users;

-- 搜索 CASE
SELECT 
    username,
    age,
    CASE
        WHEN age < 18 THEN '未成年'
        WHEN age < 65 THEN '成年人'
        ELSE '老年人'
    END AS age_group
FROM users;
```

### IF 函数（MySQL）

```sql
-- MySQL IF 函数
SELECT 
    username,
    age,
    IF(age >= 18, '成年', '未成年') AS is_adult
FROM users;

-- IFNULL 函数
SELECT 
    username,
    IFNULL(phone, '未填写') AS phone_display
FROM users;
```

### COALESCE 函数

```sql
-- 返回第一个非 NULL 值
SELECT 
    username,
    COALESCE(phone, email, '无联系方式') AS contact_info
FROM users;
```

## 0x07 集合操作

### UNION

```sql
-- 合并结果集（去重）
SELECT username FROM users
UNION
SELECT username FROM admin_users;

-- 合并结果集（保留重复）
SELECT username FROM users
UNION ALL
SELECT username FROM admin_users;
```

### INTERSECT 和 EXCEPT

```sql
-- PostgreSQL 支持
-- 交集
SELECT username FROM users
INTERSECT
SELECT username FROM admin_users;

-- 差集
SELECT username FROM users
EXCEPT
SELECT username FROM admin_users;
```

## 0x08 GROUP BY 分组查询

### 基本分组

```sql
-- 按单个字段分组
SELECT country, COUNT(*) AS user_count 
FROM users 
GROUP BY country;

-- 按多个字段分组
SELECT country, city, COUNT(*) AS user_count 
FROM users 
GROUP BY country, city;
```

### 聚合函数结合分组

```sql
-- 统计每个国家的用户数量和平均年龄
SELECT 
    country,
    COUNT(*) AS user_count,
    AVG(age) AS avg_age,
    MAX(age) AS max_age,
    MIN(age) AS min_age
FROM users 
GROUP BY country;

-- 统计每个状态的订单数量和总金额
SELECT 
    status,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_amount,
    AVG(total_amount) AS avg_amount
FROM orders 
GROUP BY status;
```

### HAVING 过滤分组

```sql
-- HAVING 用于过滤分组后的结果（WHERE 用于过滤分组前）
SELECT 
    country,
    COUNT(*) AS user_count
FROM users 
GROUP BY country
HAVING COUNT(*) > 10;

-- 组合使用：先过滤再分组再过滤
SELECT 
    department,
    AVG(salary) AS avg_salary
FROM employees
WHERE status = 'active'
GROUP BY department
HAVING AVG(salary) > 5000;
```

### 按分组内时间排序取最新记录

在实际业务中，经常需要获取每个分组内最新的记录，例如：每个用户的最新订单、每个部门的最新员工等。

#### MySQL 8.0+ 窗口函数写法

```sql
-- 使用窗口函数 ROW_NUMBER() 按时间排序取每个用户的最新订单
SELECT * FROM (
    SELECT 
        *,
        ROW_NUMBER() OVER (
            PARTITION BY user_id 
            ORDER BY created_at DESC
        ) AS rn
    FROM orders
) t
WHERE rn = 1;

-- 每个用户最近3笔订单
SELECT * FROM (
    SELECT 
        *,
        ROW_NUMBER() OVER (
            PARTITION BY user_id 
            ORDER BY created_at DESC
        ) AS rn
    FROM orders
) t
WHERE rn <= 3;
```

#### PostgreSQL 写法

```sql
-- PostgreSQL 同样支持窗口函数
SELECT * FROM (
    SELECT 
        *,
        ROW_NUMBER() OVER (
            PARTITION BY user_id 
            ORDER BY created_at DESC
        ) AS rn
    FROM orders
) t
WHERE rn = 1;

-- 使用 DISTINCT ON（PostgreSQL 特有语法）
SELECT DISTINCT ON (user_id) 
    *
FROM orders
ORDER BY user_id, created_at DESC;
```

#### 子查询写法（兼容旧版本）

```sql
-- 每个用户最新订单（子查询方式）
SELECT o.*
FROM orders o
WHERE o.created_at = (
    SELECT MAX(created_at) 
    FROM orders 
    WHERE user_id = o.user_id
);

-- 每个部门的最新入职员工
SELECT e.*
FROM employees e
WHERE e.hire_date = (
    SELECT MAX(hire_date) 
    FROM employees 
    WHERE department_id = e.department_id
);
```

#### 关联子查询取最新时间

```sql
-- 获取每个用户的最新订单时间
SELECT 
    user_id,
    (SELECT MAX(created_at) FROM orders WHERE user_id = u.user_id) AS latest_order_time
FROM users u;

-- 获取每个用户及其最新订单的完整信息（JOIN 方式）
SELECT u.*, o.*
FROM users u
LEFT JOIN (
    SELECT o1.*
    FROM orders o1
    INNER JOIN (
        SELECT user_id, MAX(created_at) AS max_date
        FROM orders
        GROUP BY user_id
    ) o2 ON o1.user_id = o2.user_id AND o1.created_at = o2.max_date
) o ON u.id = o.user_id;
```

## 参考

- [MySQL SELECT Statement](https://dev.mysql.com/doc/refman/8.0/en/select.html)
- [PostgreSQL SELECT](https://www.postgresql.org/docs/13/sql-select.html)
- [MySQL WHERE Clause](https://dev.mysql.com/doc/refman/8.0/en/where-optimizations.html)
- [PostgreSQL WHERE](https://www.postgresql.org/docs/13/queries.html)