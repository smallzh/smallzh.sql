# 基本操作

## 0x01 数据库操作

### 创建数据库

```sql
-- MySQL 创建数据库
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- PostgreSQL 创建数据库
CREATE DATABASE mydb WITH ENCODING 'UTF8' LC_COLLATE 'en_US.UTF-8';

-- 查看数据库
-- MySQL
SHOW DATABASES;

-- PostgreSQL
\l  -- psql 命令
SELECT datname FROM pg_database;
```

### 删除数据库

```sql
-- 删除数据库
DROP DATABASE mydb;

-- 如果存在则删除（PostgreSQL）
DROP DATABASE IF EXISTS mydb;
```

## 0x02 表操作（DDL）

### 创建表

```sql
-- MySQL 创建表
CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    age TINYINT CHECK (age >= 0 AND age <= 150),
    status ENUM('active', 'inactive') DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- PostgreSQL 创建表
CREATE TABLE users (
    id INTEGER GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    age SMALLINT CHECK (age >= 0 AND age <= 150),
    status VARCHAR(20) DEFAULT 'active' CHECK (status IN ('active', 'inactive')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 修改表结构

```sql
-- 添加列
-- MySQL
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- PostgreSQL
ALTER TABLE users ADD COLUMN phone VARCHAR(20);

-- 修改列类型
-- MySQL
ALTER TABLE users MODIFY COLUMN email VARCHAR(150);

-- PostgreSQL
ALTER TABLE users ALTER COLUMN email TYPE VARCHAR(150);

-- 重命名列
-- MySQL
ALTER TABLE users RENAME COLUMN phone TO mobile;

-- PostgreSQL
ALTER TABLE users RENAME COLUMN phone TO mobile;

-- 删除列
ALTER TABLE users DROP COLUMN mobile;

-- 添加约束
ALTER TABLE users ADD CONSTRAINT chk_email CHECK (email LIKE '%@%');

-- 删除约束
-- MySQL
ALTER TABLE users DROP CHECK chk_email;

-- PostgreSQL
ALTER TABLE users DROP CONSTRAINT chk_email;
```

### 删除表

```sql
-- 删除表
DROP TABLE users;

-- 如果存在则删除
DROP TABLE IF EXISTS users;

-- 删除多个表
DROP TABLE IF EXISTS users, orders, products;
```

### 查看表结构

```sql
-- MySQL
DESCRIBE users;
SHOW CREATE TABLE users;

-- PostgreSQL
\d users
\d+ users  -- 更详细的信息
```

## 0x03 数据插入（INSERT）

### 基本插入

```sql
-- 插入单行
INSERT INTO users (username, email, age) 
VALUES ('john_doe', 'john@example.com', 25);

-- 插入多行
INSERT INTO users (username, email, age) VALUES 
('jane_smith', 'jane@example.com', 30),
('bob_wilson', 'bob@example.com', 28),
('alice_brown', 'alice@example.com', 35);

-- 插入所有列（省略列名）
INSERT INTO users VALUES 
(DEFAULT, 'charlie_davis', 'charlie@example.com', 40, 'active', NOW(), NOW());
```

### 插入查询结果

```sql
-- 从另一个表插入数据
INSERT INTO users_backup (username, email, age)
SELECT username, email, age FROM users WHERE status = 'active';

-- 插入计算结果
INSERT INTO statistics (user_count, avg_age, created_at)
SELECT COUNT(*), AVG(age), NOW() FROM users;
```

### UPSERT 操作

```sql
-- MySQL: INSERT ... ON DUPLICATE KEY UPDATE
INSERT INTO users (username, email, age) 
VALUES ('john_doe', 'newemail@example.com', 26)
ON DUPLICATE KEY UPDATE 
    email = VALUES(email),
    age = VALUES(age);

-- PostgreSQL: INSERT ... ON CONFLICT
INSERT INTO users (username, email, age) 
VALUES ('john_doe', 'newemail@example.com', 26)
ON CONFLICT (username) DO UPDATE SET 
    email = EXCLUDED.email,
    age = EXCLUDED.age;
```

## 0x04 数据更新（UPDATE）

### 基本更新

```sql
-- 更新单列
UPDATE users SET age = 26 WHERE username = 'john_doe';

-- 更新多列
UPDATE users SET 
    age = 26, 
    email = 'newemail@example.com',
    updated_at = NOW()
WHERE username = 'john_doe';
```

### 条件更新

```sql
-- 根据条件更新
UPDATE users SET status = 'inactive' 
WHERE last_login < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- 使用 CASE 语句
UPDATE users SET 
    status = CASE 
        WHEN age < 18 THEN 'minor'
        WHEN age < 65 THEN 'adult'
        ELSE 'senior'
    END;
```

### 更新关联表

```sql
-- MySQL 多表更新
UPDATE users u 
JOIN orders o ON u.id = o.user_id 
SET u.total_orders = o.order_count;

-- PostgreSQL 多表更新
UPDATE users u 
SET total_orders = o.order_count
FROM orders o 
WHERE u.id = o.user_id;
```

## 0x05 数据删除（DELETE）

### 基本删除

```sql
-- 删除特定行
DELETE FROM users WHERE username = 'john_doe';

-- 删除多行
DELETE FROM users WHERE status = 'inactive' AND age > 60;

-- 删除所有数据（保留表结构）
DELETE FROM users;
```

### 清空表

```sql
-- DELETE: 逐行删除，记录日志
DELETE FROM users;

-- TRUNCATE: 快速清空，重置自增
TRUNCATE TABLE users;

-- DROP + CREATE: 完全重建
DROP TABLE users;
CREATE TABLE users (...);
```

### 级联删除

```sql
-- 创建带级联删除的外键
CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);

-- 删除用户时，相关订单会自动删除
DELETE FROM users WHERE id = 1;
```

## 0x06 事务控制

### 基本事务

```sql
-- 开始事务
START TRANSACTION;  -- MySQL
BEGIN;              -- PostgreSQL/通用

-- 执行操作
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- 提交事务
COMMIT;

-- 回滚事务
ROLLBACK;
```

### 保存点

```sql
BEGIN;

UPDATE accounts SET balance = balance - 100 WHERE id = 1;
SAVEPOINT point1;

UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- 如果出错，回滚到保存点
ROLLBACK TO point1;

-- 继续其他操作
UPDATE accounts SET balance = balance + 100 WHERE id = 3;
COMMIT;
```

## 参考

- [MySQL DDL Statements](https://dev.mysql.com/doc/refman/8.0/en/sql-data-definition-statements.html)
- [MySQL DML Statements](https://dev.mysql.com/doc/refman/8.0/en/sql-data-manipulation-statements.html)
- [PostgreSQL DDL](https://www.postgresql.org/docs/13/ddl.html)
- [PostgreSQL DML](https://www.postgresql.org/docs/13/dml.html)