# 高级查询

## 0x01 JOIN 连接查询

### INNER JOIN（内连接）

只返回两个表中都匹配的行：

```sql
-- 基本内连接
SELECT 
    u.username,
    o.order_id,
    o.total_amount
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- 多表连接
SELECT 
    u.username,
    o.order_id,
    p.product_name,
    oi.quantity
FROM users u
INNER JOIN orders o ON u.id = o.user_id
INNER JOIN order_items oi ON o.id = oi.order_id
INNER JOIN products p ON oi.product_id = p.id;
```

### LEFT JOIN（左连接）

返回左表所有行，右表不匹配的显示 NULL：

```sql
-- 查询所有用户及其订单（包括没有订单的用户）
SELECT 
    u.username,
    o.order_id,
    o.total_amount
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;

-- 查找没有订单的用户
SELECT 
    u.username
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;
```

### RIGHT JOIN（右连接）

返回右表所有行，左表不匹配的显示 NULL：

```sql
-- 查询所有订单及其用户
SELECT 
    u.username,
    o.order_id
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;
```

### FULL JOIN（全连接）

返回两个表的所有行：

```sql
-- PostgreSQL 支持全连接
SELECT 
    u.username,
    o.order_id
FROM users u
FULL JOIN orders o ON u.id = o.user_id;

-- MySQL 不支持 FULL JOIN，可用 UNION 模拟
SELECT u.username, o.order_id FROM users u LEFT JOIN orders o ON u.id = o.user_id
UNION
SELECT u.username, o.order_id FROM users u RIGHT JOIN orders o ON u.id = o.user_id;
```

### CROSS JOIN（交叉连接）

返回两个表的笛卡尔积：

```sql
-- 交叉连接（所有组合）
SELECT 
    u.username,
    p.product_name
FROM users u
CROSS JOIN products p;
```

### SELF JOIN（自连接）

表与自身连接：

```sql
-- 查找员工及其经理
SELECT 
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

## 0x02 子查询

### WHERE 子查询

```sql
-- 标量子查询
SELECT * FROM users WHERE age = (SELECT MAX(age) FROM users);

-- 列子查询
SELECT * FROM users WHERE id IN (SELECT user_id FROM orders);

-- NOT IN 子查询
SELECT * FROM users WHERE id NOT IN (SELECT user_id FROM orders);

-- EXISTS 子查询
SELECT * FROM users u 
WHERE EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);

-- NOT EXISTS 子查询
SELECT * FROM users u 
WHERE NOT EXISTS (SELECT 1 FROM orders o WHERE o.user_id = u.id);
```

### FROM 子查询

```sql
-- 派生表
SELECT 
    avg_data.age_group,
    avg_data.avg_amount
FROM (
    SELECT 
        CASE 
            WHEN u.age < 25 THEN 'young'
            WHEN u.age < 45 THEN 'middle'
            ELSE 'senior'
        END AS age_group,
        AVG(o.total_amount) AS avg_amount
    FROM users u
    JOIN orders o ON u.id = o.user_id
    GROUP BY age_group
) AS avg_data
WHERE avg_data.avg_amount > 100;
```

### SELECT 子查询

```sql
-- 相关子查询
SELECT 
    u.username,
    u.age,
    (SELECT COUNT(*) FROM orders o WHERE o.user_id = u.id) AS order_count,
    (SELECT MAX(o.total_amount) FROM orders o WHERE o.user_id = u.id) AS max_order
FROM users u;
```

## 0x03 聚合函数

### 常用聚合函数

```sql
-- COUNT: 计数
SELECT COUNT(*) FROM users;  -- 总行数
SELECT COUNT(phone) FROM users;  -- 非 NULL 行数
SELECT COUNT(DISTINCT country) FROM users;  -- 不重复计数

-- SUM: 求和
SELECT SUM(total_amount) FROM orders;

-- AVG: 平均值
SELECT AVG(age) FROM users;

-- MAX: 最大值
SELECT MAX(age) FROM users;
SELECT MAX(created_at) FROM users;  -- 最新日期

-- MIN: 最小值
SELECT MIN(age) FROM users;
SELECT MIN(created_at) FROM users;  -- 最早日期
```

### GROUP BY 分组

```sql
-- 按国家分组统计
SELECT 
    country,
    COUNT(*) AS user_count,
    AVG(age) AS avg_age
FROM users
GROUP BY country;

-- 多列分组
SELECT 
    country,
    city,
    COUNT(*) AS user_count
FROM users
GROUP BY country, city;
```

### HAVING 过滤分组

```sql
-- HAVING 过滤分组结果
SELECT 
    country,
    COUNT(*) AS user_count,
    AVG(age) AS avg_age
FROM users
GROUP BY country
HAVING COUNT(*) > 100 AND AVG(age) > 25;

-- WHERE vs HAVING
-- WHERE: 分组前过滤（不能使用聚合函数）
-- HAVING: 分组后过滤（可以使用聚合函数）

SELECT 
    country,
    COUNT(*) AS user_count
FROM users
WHERE age >= 18  -- 分组前过滤
GROUP BY country
HAVING COUNT(*) > 50;  -- 分组后过滤
```

## 0x04 窗口函数

### 基本窗口函数

```sql
-- ROW_NUMBER: 行号
SELECT 
    username,
    age,
    ROW_NUMBER() OVER (ORDER BY age DESC) AS row_num
FROM users;

-- RANK: 排名（有并列）
SELECT 
    username,
    age,
    RANK() OVER (ORDER BY age DESC) AS rank
FROM users;

-- DENSE_RANK: 密集排名（无间隔）
SELECT 
    username,
    age,
    DENSE_RANK() OVER (ORDER BY age DESC) AS dense_rank
FROM users;
```

### 分区窗口函数

```sql
-- 按国家分组排名
SELECT 
    username,
    country,
    age,
    ROW_NUMBER() OVER (PARTITION BY country ORDER BY age DESC) AS country_rank
FROM users;

-- 每个国家年龄最大的用户
WITH ranked_users AS (
    SELECT 
        username,
        country,
        age,
        ROW_NUMBER() OVER (PARTITION BY country ORDER BY age DESC) AS rn
    FROM users
)
SELECT * FROM ranked_users WHERE rn = 1;
```

### 聚合窗口函数

```sql
-- 累计求和
SELECT 
    order_id,
    order_date,
    total_amount,
    SUM(total_amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- 移动平均
SELECT 
    order_id,
    order_date,
    total_amount,
    AVG(total_amount) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg
FROM orders;

-- 分组聚合
SELECT 
    username,
    country,
    age,
    AVG(age) OVER (PARTITION BY country) AS country_avg_age,
    age - AVG(age) OVER (PARTITION BY country) AS age_diff
FROM users;
```

### LAG 和 LEAD 函数

```sql
-- LAG: 获取前一行的值
SELECT 
    order_id,
    order_date,
    total_amount,
    LAG(total_amount) OVER (ORDER BY order_date) AS prev_amount
FROM orders;

-- LEAD: 获取后一行的值
SELECT 
    order_id,
    order_date,
    total_amount,
    LEAD(total_amount) OVER (ORDER BY order_date) AS next_amount
FROM orders;

-- 计算与前一天的差额
SELECT 
    order_id,
    order_date,
    total_amount,
    total_amount - LAG(total_amount) OVER (ORDER BY order_date) AS amount_diff
FROM orders;
```

## 0x05 CTE 公用表表达式

### 基本 CTE

```sql
-- 非递归 CTE
WITH user_stats AS (
    SELECT 
        user_id,
        COUNT(*) AS order_count,
        SUM(total_amount) AS total_spent
    FROM orders
    GROUP BY user_id
)
SELECT 
    u.username,
    us.order_count,
    us.total_spent
FROM users u
JOIN user_stats us ON u.id = us.user_id;
```

### 递归 CTE

```sql
-- 生成数字序列
WITH RECURSIVE numbers AS (
    SELECT 1 AS n
    UNION ALL
    SELECT n + 1 FROM numbers WHERE n < 100
)
SELECT * FROM numbers;

-- 组织架构树
WITH RECURSIVE org_tree AS (
    -- 基础查询：顶级管理者
    SELECT id, name, manager_id, 1 AS level
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- 递归查询：下属员工
    SELECT e.id, e.name, e.manager_id, ot.level + 1
    FROM employees e
    JOIN org_tree ot ON e.manager_id = ot.id
)
SELECT * FROM org_tree ORDER BY level, name;
```

## 0x06 高级查询技巧

### PIVOT（行转列）

```sql
-- MySQL 使用 CASE 语句实现
SELECT 
    product_id,
    SUM(CASE WHEN quarter = 'Q1' THEN sales END) AS Q1,
    SUM(CASE WHEN quarter = 'Q2' THEN sales END) AS Q2,
    SUM(CASE WHEN quarter = 'Q3' THEN sales END) AS Q3,
    SUM(CASE WHEN quarter = 'Q4' THEN sales END) AS Q4
FROM sales
GROUP BY product_id;

-- PostgreSQL 使用 crosstab 函数
SELECT * FROM crosstab(
    'SELECT product_id, quarter, sales FROM sales ORDER BY 1,2',
    'SELECT DISTINCT quarter FROM sales ORDER BY 1'
) AS ct(product_id INT, Q1 INT, Q2 INT, Q3 INT, Q4 INT);
```

### UNPIVOT（列转行）

```sql
-- 使用 UNION ALL 实现
SELECT product_id, 'Q1' AS quarter, Q1 AS sales FROM sales
UNION ALL
SELECT product_id, 'Q2', Q2 FROM sales
UNION ALL
SELECT product_id, 'Q3', Q3 FROM sales
UNION ALL
SELECT product_id, 'Q4', Q4 FROM sales;
```

### 字符串聚合

```sql
-- MySQL: GROUP_CONCAT
SELECT 
    department,
    GROUP_CONCAT(employee_name ORDER BY employee_name SEPARATOR ', ') AS employees
FROM employees
GROUP BY department;

-- PostgreSQL: STRING_AGG
SELECT 
    department,
    STRING_AGG(employee_name, ', ' ORDER BY employee_name) AS employees
FROM employees
GROUP BY department;
```

## 参考

- [MySQL JOIN Syntax](https://dev.mysql.com/doc/refman/8.0/en/join.html)
- [PostgreSQL Table Expressions](https://www.postgresql.org/docs/13/queries-table-expressions.html)
- [MySQL Window Functions](https://dev.mysql.com/doc/refman/8.0/en/window-functions.html)
- [PostgreSQL Window Functions](https://www.postgresql.org/docs/13/tutorial-window.html)