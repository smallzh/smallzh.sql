# 常用函数

## 0x01 字符串函数

### 基本字符串操作

```sql
-- LENGTH: 字符串长度
SELECT LENGTH('Hello World');  -- 结果: 11

-- CHAR_LENGTH: 字符长度（多字节字符）
SELECT CHAR_LENGTH('你好');  -- 结果: 2

-- CONCAT: 字符串连接
SELECT CONCAT('Hello', ' ', 'World');  -- 结果: 'Hello World'

-- MySQL 支持多个参数
SELECT CONCAT(first_name, ' ', last_name) AS full_name FROM users;

-- PostgreSQL 使用 || 连接
SELECT first_name || ' ' || last_name AS full_name FROM users;
```

### 字符串提取

```sql
-- SUBSTRING/SUBSTR: 提取子串
SELECT SUBSTRING('Hello World', 1, 5);  -- 结果: 'Hello'
SELECT SUBSTR('Hello World', 7);  -- 结果: 'World'

-- LEFT/RIGHT: 从左/右提取
SELECT LEFT('Hello', 3);  -- 结果: 'Hel'
SELECT RIGHT('Hello', 3);  -- 结果: 'llo'
```

### 字符串查找和替换

```sql
-- LOCATE/POSITION: 查找位置
-- MySQL
SELECT LOCATE('World', 'Hello World');  -- 结果: 7
-- PostgreSQL
SELECT POSITION('World' IN 'Hello World');  -- 结果: 7

-- REPLACE: 替换
SELECT REPLACE('Hello World', 'World', 'SQL');  -- 结果: 'Hello SQL'

-- INSTR: 查找位置（MySQL）
SELECT INSTR('Hello World', 'World');  -- 结果: 7
```

### 字符串转换

```sql
-- 大小写转换
SELECT UPPER('hello');  -- 结果: 'HELLO'
SELECT LOWER('HELLO');  -- 结果: 'hello'

-- 去除空格
SELECT TRIM('  Hello  ');  -- 去除两端空格
SELECT LTRIM('  Hello');   -- 去除左端空格
SELECT RTRIM('Hello  ');   -- 去除右端空格

-- 填充
SELECT LPAD('5', 3, '0');  -- 结果: '005'
SELECT RPAD('5', 3, '0');  -- 结果: '500'
```

### 字符串分割

```sql
-- MySQL: SUBSTRING_INDEX
SELECT SUBSTRING_INDEX('www.example.com', '.', 1);   -- 结果: 'www'
SELECT SUBSTRING_INDEX('www.example.com', '.', -1);  -- 结果: 'com'

-- PostgreSQL: SPLIT_PART
SELECT SPLIT_PART('www.example.com', '.', 1);  -- 结果: 'www'
SELECT SPLIT_PART('www.example.com', '.', 3);  -- 结果: 'com'
```

## 0x02 数值函数

### 基本数值操作

```sql
-- ABS: 绝对值
SELECT ABS(-10);  -- 结果: 10

-- CEIL/CEILING: 向上取整
SELECT CEIL(4.2);  -- 结果: 5
SELECT CEILING(4.2);  -- 结果: 5

-- FLOOR: 向下取整
SELECT FLOOR(4.8);  -- 结果: 4

-- ROUND: 四舍五入
SELECT ROUND(4.5);  -- 结果: 5
SELECT ROUND(4.4);  -- 结果: 4
SELECT ROUND(4.567, 2);  -- 结果: 4.57
```

### 随机数和幂运算

```sql
-- RAND/RANDOM: 随机数
-- MySQL
SELECT RAND();  -- 0到1之间的随机数
SELECT RAND() * 100;  -- 0到100之间的随机数
SELECT FLOOR(RAND() * 100);  -- 0到99之间的随机整数

-- PostgreSQL
SELECT RANDOM();  -- 0到1之间的随机数

-- POW/POWER: 幂运算
SELECT POW(2, 3);  -- 结果: 8
SELECT POWER(2, 3);  -- 结果: 8

-- SQRT: 平方根
SELECT SQRT(16);  -- 结果: 4
```

### 数值格式化

```sql
-- TRUNCATE: 截断小数
SELECT TRUNCATE(4.567, 2);  -- 结果: 4.56

-- FORMAT: 格式化数字（MySQL）
SELECT FORMAT(1234567.89, 2);  -- 结果: '1,234,567.89'

-- TO_CHAR: 格式化数字（PostgreSQL）
SELECT TO_CHAR(1234567.89, '9,999,999.99');  -- 结果: '1,234,567.89'
```

## 0x03 日期时间函数

### 获取当前时间

```sql
-- 当前日期时间
SELECT NOW();  -- MySQL/PostgreSQL
SELECT CURRENT_TIMESTAMP;  -- 标准SQL

-- 当前日期
SELECT CURRENT_DATE;
SELECT CURDATE();  -- MySQL

-- 当前时间
SELECT CURRENT_TIME;
SELECT CURTIME();  -- MySQL
```

### 日期时间提取

```sql
-- MySQL: EXTRACT
SELECT EXTRACT(YEAR FROM NOW());  -- 年
SELECT EXTRACT(MONTH FROM NOW());  -- 月
SELECT EXTRACT(DAY FROM NOW());  -- 日
SELECT EXTRACT(HOUR FROM NOW());  -- 时

-- MySQL: YEAR/MONTH/DAY
SELECT YEAR(NOW());
SELECT MONTH(NOW());
SELECT DAY(NOW());

-- PostgreSQL: EXTRACT 和 DATE_PART
SELECT EXTRACT(YEAR FROM NOW());
SELECT DATE_PART('year', NOW());

-- DAYNAME/MONTHNAME
-- MySQL
SELECT DAYNAME(NOW());  -- 星期几
SELECT MONTHNAME(NOW());  -- 月份名称
```

### 日期时间计算

```sql
-- MySQL: DATE_ADD/DATE_SUB
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY);  -- 加1天
SELECT DATE_ADD(NOW(), INTERVAL 1 MONTH);  -- 加1个月
SELECT DATE_ADD(NOW(), INTERVAL -1 HOUR);  -- 减1小时

SELECT DATE_SUB(NOW(), INTERVAL 1 WEEK);  -- 减1周

-- PostgreSQL: 直接加减
SELECT NOW() + INTERVAL '1 day';  -- 加1天
SELECT NOW() + INTERVAL '1 month';  -- 加1个月
SELECT NOW() - INTERVAL '1 hour';  -- 减1小时

-- 日期差计算
-- MySQL
SELECT DATEDIFF('2024-12-31', '2024-01-01');  -- 天数差
SELECT TIMESTAMPDIFF(HOUR, '2024-01-01 10:00:00', '2024-01-02 12:00:00');  -- 小时差

-- PostgreSQL
SELECT '2024-12-31'::DATE - '2024-01-01'::DATE;  -- 天数差
SELECT '2024-01-02 12:00:00'::TIMESTAMP - '2024-01-01 10:00:00'::TIMESTAMP;  -- 时间差
```

### 日期时间格式化

```sql
-- MySQL: DATE_FORMAT
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d');  -- 2024-01-01
SELECT DATE_FORMAT(NOW(), '%Y年%m月%d日 %H:%i:%s');  -- 2024年01月01日 12:00:00

-- PostgreSQL: TO_CHAR
SELECT TO_CHAR(NOW(), 'YYYY-MM-DD');  -- 2024-01-01
SELECT TO_CHAR(NOW(), 'YYYY年MM月DD日 HH24:MI:SS');  -- 2024年01月01日 12:00:00

-- 日期解析
-- MySQL: STR_TO_DATE
SELECT STR_TO_DATE('2024-01-01', '%Y-%m-%d');

-- PostgreSQL: TO_DATE
SELECT TO_DATE('2024-01-01', 'YYYY-MM-DD');
```

### 日期时间常用操作

```sql
-- 获取月初/月末
-- MySQL
SELECT DATE_FORMAT(NOW(), '%Y-%m-01');  -- 月初
SELECT LAST_DAY(NOW());  -- 月末

-- PostgreSQL
SELECT DATE_TRUNC('month', NOW());  -- 月初
SELECT (DATE_TRUNC('month', NOW()) + INTERVAL '1 month - 1 day')::DATE;  -- 月末

-- 获取季度
SELECT QUARTER(NOW());  -- MySQL
SELECT EXTRACT(QUARTER FROM NOW());  -- PostgreSQL

-- 获取星期几（1=周日，7=周六）
SELECT DAYOFWEEK(NOW());  -- MySQL
SELECT EXTRACT(DOW FROM NOW());  -- PostgreSQL (0=周日)
```

## 0x04 条件函数

### IF 函数

```sql
-- MySQL IF
SELECT IF(age >= 18, '成年', '未成年') AS age_status FROM users;

-- PostgreSQL 使用 CASE 替代
SELECT CASE WHEN age >= 18 THEN '成年' ELSE '未成年' END AS age_status FROM users;
```

### COALESCE 函数

```sql
-- 返回第一个非 NULL 值
SELECT COALESCE(phone, email, '无联系方式') AS contact FROM users;

-- 常用于默认值
SELECT COALESCE(nickname, username) AS display_name FROM users;
```

### NULLIF 函数

```sql
-- 如果两个值相等则返回 NULL
SELECT NULLIF(10, 10);  -- 结果: NULL
SELECT NULLIF(10, 20);  -- 结果: 10

-- 常用于避免除零错误
SELECT 100 / NULLIF(quantity, 0) AS unit_price FROM products;
```

### CASE 表达式

```sql
-- 简单 CASE
SELECT 
    CASE status
        WHEN 'active' THEN '活跃'
        WHEN 'inactive' THEN '非活跃'
        ELSE '未知'
    END AS status_text
FROM users;

-- 搜索 CASE
SELECT 
    CASE
        WHEN age < 18 THEN '未成年'
        WHEN age < 40 THEN '青年'
        WHEN age < 60 THEN '中年'
        ELSE '老年'
    END AS age_group
FROM users;
```

## 0x05 类型转换函数

### CAST 函数

```sql
-- 标准 SQL CAST
SELECT CAST('123' AS INTEGER);  -- 字符串转整数
SELECT CAST(123 AS CHAR);  -- 整数转字符串
SELECT CAST('2024-01-01' AS DATE);  -- 字符串转日期

-- MySQL 特有语法
SELECT CAST('123' AS SIGNED);  -- 转为有符号整数
SELECT CAST('123' AS UNSIGNED);  -- 转为无符号整数
```

### PostgreSQL 类型转换

```sql
-- 使用 :: 操作符
SELECT '123'::INTEGER;  -- 字符串转整数
SELECT 123::TEXT;  -- 整数转字符串
SELECT '2024-01-01'::DATE;  -- 字符串转日期
SELECT '2024-01-01 12:00:00'::TIMESTAMP;  -- 字符串转时间戳
```

### 转换函数

```sql
-- MySQL: CONVERT
SELECT CONVERT('你好' USING utf8mb4);  -- 字符集转换

-- PostgreSQL: TO_NUMBER, TO_DATE, TO_CHAR
SELECT TO_NUMBER('1234.56', '9999.99');  -- 字符串转数字
SELECT TO_DATE('2024-01-01', 'YYYY-MM-DD');  -- 字符串转日期
SELECT TO_CHAR(1234.56, '9,999.99');  -- 数字转格式化字符串
```

## 0x06 JSON 函数

### MySQL JSON 函数

```sql
-- 创建 JSON
SELECT JSON_OBJECT('name', 'John', 'age', 30);
SELECT JSON_ARRAY(1, 2, 3);

-- 提取 JSON 值
SELECT JSON_EXTRACT('{"name": "John", "age": 30}', '$.name');  -- 结果: 'John'
SELECT '{"name": "John"}'->>'$.name';  -- 简写形式

-- JSON 操作
SELECT JSON_SET('{"name": "John"}', '$.age', 30);  -- 设置值
SELECT JSON_REMOVE('{"name": "John", "age": 30}', '$.age');  -- 删除键
```

### PostgreSQL JSON 函数

```sql
-- 创建 JSON
SELECT json_build_object('name', 'John', 'age', 30);
SELECT json_build_array(1, 2, 3);

-- 提取 JSON 值
SELECT '{"name": "John"}'::json->>'name';  -- 结果: 'John'
SELECT '{"user": {"name": "John"}}'::json->'user'->>'name';  -- 嵌套提取

-- JSONB 操作（更高效）
SELECT '{"name": "John", "age": 30}'::jsonb - 'age';  -- 删除键
SELECT '{"name": "John"}'::jsonb || '{"age": 30}'::jsonb;  -- 合并
```

## 参考

- [MySQL Functions](https://dev.mysql.com/doc/refman/8.0/en/functions.html)
- [PostgreSQL Functions](https://www.postgresql.org/docs/13/functions.html)
- [MySQL Date Functions](https://dev.mysql.com/doc/refman/8.0/en/date-and-time-functions.html)
- [PostgreSQL Date Functions](https://www.postgresql.org/docs/13/functions-datetime.html)