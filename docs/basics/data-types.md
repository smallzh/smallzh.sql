# 数据类型

## 0x01 数值类型

### 整数类型

| 类型 | MySQL | PostgreSQL | 存储大小 | 说明 |
|------|-------|------------|----------|------|
| TINYINT | ✓ | ✗ | 1字节 | 小整数 (-128 到 127) |
| SMALLINT | ✓ | ✓ | 2字节 | 小整数 (-32768 到 32767) |
| INT/INTEGER | ✓ | ✓ | 4字节 | 标准整数 |
| BIGINT | ✓ | ✓ | 8字节 | 大整数 |

```sql
-- MySQL 示例
CREATE TABLE numeric_examples (
    id INT PRIMARY KEY,
    age TINYINT,
    count SMALLINT,
    population BIGINT
);

-- PostgreSQL 示例
CREATE TABLE numeric_examples (
    id INTEGER PRIMARY KEY,
    age SMALLINT,
    count INTEGER,
    population BIGINT
);
```

### 小数类型

| 类型 | MySQL | PostgreSQL | 说明 |
|------|-------|------------|------|
| DECIMAL(p,s) | ✓ | ✓ | 精确小数，p=总位数，s=小数位数 |
| NUMERIC(p,s) | ✓ | ✓ | DECIMAL 的同义词 |
| FLOAT | ✓ | ✓ | 单精度浮点数 |
| DOUBLE | ✓ | ✗ | 双精度浮点数 |
| REAL | ✓ | ✓ | 单精度浮点数 |

```sql
-- 精确数值类型示例
CREATE TABLE financial_data (
    id INT PRIMARY KEY,
    price DECIMAL(10,2),      -- 总共10位，其中2位小数
    tax_rate DECIMAL(5,4),    -- 税率，如 0.1500
    quantity INT
);

-- 插入数据
INSERT INTO financial_data VALUES (1, 1234.56, 0.1500, 10);
```

### 自增类型

```sql
-- MySQL 自增
CREATE TABLE auto_increment_example (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50)
);

-- PostgreSQL 自增 (推荐使用 IDENTITY)
CREATE TABLE auto_increment_example (
    id SERIAL PRIMARY KEY,           -- PostgreSQL 传统方式
    name VARCHAR(50)
);

-- PostgreSQL 13+ 标准方式
CREATE TABLE auto_increment_example (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    name VARCHAR(50)
);
```

## 0x02 字符串类型

### 定长与变长字符串

| 类型 | MySQL | PostgreSQL | 说明 |
|------|-------|------------|------|
| CHAR(n) | ✓ | ✓ | 定长字符串，不足补空格 |
| VARCHAR(n) | ✓ | ✓ | 变长字符串，最大长度n |
| TEXT | ✓ | ✓ | 长文本数据 |

```sql
-- 字符串类型示例
CREATE TABLE string_examples (
    id INT PRIMARY KEY,
    country_code CHAR(2),           -- 国家代码，固定2位
    username VARCHAR(50),           -- 用户名，变长
    bio TEXT,                       -- 个人简介
    email VARCHAR(100)              -- 邮箱
);

-- 插入数据
INSERT INTO string_examples VALUES 
(1, 'CN', '张三', '热爱编程的开发者', 'zhangsan@example.com'),
(2, 'US', 'john_doe', 'Software engineer', 'john@example.com');
```

### 特殊字符串类型

```sql
-- MySQL 特有类型
CREATE TABLE mysql_string_examples (
    id INT PRIMARY KEY,
    status ENUM('active', 'inactive', 'pending'),  -- 枚举类型
    tags SET('tag1', 'tag2', 'tag3')               -- 集合类型
);

-- PostgreSQL 特有类型
CREATE TABLE postgresql_string_examples (
    id INTEGER PRIMARY KEY,
    data JSON,                                     -- JSON 数据
    metadata JSONB,                                -- 二进制 JSON
    ip_address INET,                               -- IP 地址
    mac_address MACADDR                            -- MAC 地址
);
```

## 0x03 日期时间类型

### 日期时间类型

| 类型 | MySQL | PostgreSQL | 说明 |
|------|-------|------------|------|
| DATE | ✓ | ✓ | 日期 |
| TIME | ✓ | ✓ | 时间 |
| DATETIME | ✓ | ✗ | 日期时间 |
| TIMESTAMP | ✓ | ✓ | 时间戳 |
| YEAR | ✓ | ✗ | 年份 |

```sql
-- MySQL 日期时间示例
CREATE TABLE datetime_examples (
    id INT PRIMARY KEY,
    birth_date DATE,                    -- 出生日期
    created_at DATETIME,                -- 创建时间
    updated_at TIMESTAMP,               -- 更新时间
    event_time TIME,                    -- 事件时间
    birth_year YEAR                     -- 出生年份
);

-- PostgreSQL 日期时间示例
CREATE TABLE datetime_examples (
    id INTEGER PRIMARY KEY,
    birth_date DATE,                    -- 出生日期
    event_time TIME,                    -- 时间
    created_at TIMESTAMP,               -- 时间戳
    updated_at TIMESTAMPTZ,             -- 带时区的时间戳
    duration INTERVAL                   -- 时间间隔
);
```

### 日期时间函数示例

```sql
-- 获取当前时间
SELECT NOW();                          -- MySQL/PostgreSQL
SELECT CURRENT_DATE;                   -- 当前日期
SELECT CURRENT_TIME;                   -- 当前时间

-- 日期格式化
-- MySQL
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');

-- PostgreSQL
SELECT TO_CHAR(NOW(), 'YYYY-MM-DD HH24:MI:SS');

-- 日期计算
-- MySQL
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY);    -- 加1天
SELECT DATE_SUB(NOW(), INTERVAL 1 MONTH);  -- 减1个月

-- PostgreSQL
SELECT NOW() + INTERVAL '1 day';           -- 加1天
SELECT NOW() - INTERVAL '1 month';         -- 减1个月
```

## 0x04 二进制类型

| 类型 | MySQL | PostgreSQL | 说明 |
|------|-------|------------|------|
| BLOB | ✓ | ✗ | 二进制大对象 |
| BYTEA | ✗ | ✓ | 二进制数据 |
| BINARY(n) | ✓ | ✗ | 固定长度二进制 |
| VARBINARY(n) | ✓ | ✗ | 可变长度二进制 |

```sql
-- MySQL 二进制示例
CREATE TABLE binary_examples (
    id INT PRIMARY KEY,
    image BLOB,                         -- 图片数据
    hash BINARY(32),                    -- 固定长度哈希值
    token VARBINARY(255)                -- 可变长度令牌
);

-- PostgreSQL 二进制示例
CREATE TABLE binary_examples (
    id INTEGER PRIMARY KEY,
    image BYTEA,                        -- 二进制数据
    hash BYTEA                          -- 哈希值
);
```

## 0x05 布尔类型

```sql
-- MySQL 布尔类型（实际上是 TINYINT(1) 的别名）
CREATE TABLE boolean_examples (
    id INT PRIMARY KEY,
    is_active BOOLEAN,                  -- 等同于 TINYINT(1)
    is_deleted BOOL                     -- 同上
);

-- PostgreSQL 布尔类型
CREATE TABLE boolean_examples (
    id INTEGER PRIMARY KEY,
    is_active BOOLEAN,                  -- 真正的布尔类型
    is_verified BOOL                    -- BOOLEAN 的别名
);

-- 布尔值使用
INSERT INTO boolean_examples VALUES (1, TRUE, FALSE);
INSERT INTO boolean_examples VALUES (2, 1, 0);      -- MySQL 中 1=true, 0=false
```

## 0x06 类型转换

### 隐式转换

数据库会自动进行某些类型转换：

```sql
-- 字符串转数字（自动转换）
SELECT '123' + 1;  -- 结果: 124

-- 数字转字符串（自动转换）
SELECT CONCAT('ID: ', 123);  -- 结果: 'ID: 123'
```

### 显式转换

```sql
-- MySQL 转换函数
SELECT CAST('123' AS SIGNED);           -- 转换为整数
SELECT CONVERT('2024-01-01', DATE);     -- 转换为日期

-- PostgreSQL 转换函数
SELECT CAST('123' AS INTEGER);          -- 转换为整数
SELECT '123'::INTEGER;                  -- PostgreSQL 特有语法
SELECT TO_DATE('2024-01-01', 'YYYY-MM-DD');  -- 转换为日期
```

## 参考

- [MySQL Data Types](https://dev.mysql.com/doc/refman/8.0/en/data-types.html)
- [PostgreSQL Data Types](https://www.postgresql.org/docs/13/datatype.html)