# MySQL8 / PostgreSQL13 如何导入外部SQL文件？

> 本文介绍在 MySQL 8 和 PostgreSQL 13 中导入外部 SQL 文件的几种常用方法，包括包含建表语句和插入数据的 SQL 文件。

## 0x01 前置准备

在导入 SQL 文件之前，需要确保：

- MySQL 8 服务已启动
- 已创建目标数据库（如果 SQL 文件中不包含创建数据库的语句）
- 拥有数据库的操作权限

```sql
-- 登录 MySQL
mysql -u root -p

-- 创建目标数据库（如果需要）
CREATE DATABASE IF NOT EXISTS mydb;
USE mydb;
```

## 0x02 使用 source 命令导入

`source` 是 MySQL 客户端内置的命令，适合导入中小型的 SQL 文件。

### 基本语法

```sql
SOURCE /path/to/yourfile.sql;
```

### 示例

```sql
-- 在 MySQL 客户端中执行
SOURCE /home/user/data/init.sql;

-- Windows 路径示例（注意双反斜杠）
SOURCE C:\\Users\\admin\\data\\init.sql;
```

### 注意事项

- 文件路径可以是绝对路径或相对路径（相对于 MySQL 客户端启动目录）
- 如果 SQL 文件包含 `USE database_name;` 语句，可以省略先前的 `USE` 命令
- 大文件导入可能耗时较长，耐心等待直到出现 `Query OK` 提示

## 0x03 使用 mysql 命令行导入

在操作系统命令行直接执行，适合批量导入或脚本自动化。

### 基本语法

```bash
mysql -u username -p database_name < /path/to/sqlfile.sql
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `-u` | 指定用户名 |
| `-p` | 提示输入密码 |
| `database_name` | 目标数据库名 |
| `<` | 输入重定向符号 |

### 示例

```bash
# 导入到指定数据库
mysql -u root -p mydb < init.sql

# 如果 SQL 文件包含创建数据库语句
mysql -u root -p < full_backup.sql

# 指定字符集（解决中文乱码问题）
mysql -u root -p --default-character-set=utf8mb4 mydb < init.sql
```

## 0x04 导入包含建表和数据的 SQL 文件

下面是一个完整的示例 SQL 文件，包含了建表语句和插入数据：

```sql
-- init_database.sql

-- 创建数据库
CREATE DATABASE IF NOT EXISTS shop CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE shop;

-- 创建用户表
CREATE TABLE IF NOT EXISTS users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 创建商品表
CREATE TABLE IF NOT EXISTS products (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock INT DEFAULT 0,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- 插入用户数据
INSERT INTO users (username, email, password_hash) VALUES
('admin', 'admin@example.com', '$2b$10$abcdefghijklmnopqrstuv'),
('user1', 'user1@example.com', '$2b$10$zyxwvutsrqponmlkjihgf'),
('user2', 'user2@example.com', '$2b$10$ABCDEFGHIJKLMNOPQRSTU');

-- 插入商品数据
INSERT INTO products (name, description, price, stock, category_id) VALUES
('笔记本电脑', '15.6英寸轻薄本', 5999.00, 50, 1),
('无线鼠标', '蓝牙5.0无线鼠标', 89.00, 200, 2),
('机械键盘', '青轴机械键盘', 299.00, 80, 2);
```

### 导入示例

```bash
# 方法一：使用 mysql 命令行
mysql -u root -p shop < init_database.sql

# 方法二：在 MySQL 客户端内
mysql> SOURCE /path/to/init_database.sql;

# 方法三：指定字符集（推荐）
mysql -u root -p --default-character-set=utf8mb4 shop < init_database.sql
```

## 0x05 导入大文件的注意事项

### 使用命令行导入

对于超过几十 MB 的大文件，建议使用操作系统命令行：

```bash
# 启用详细输出，查看导入进度
mysql -u root -p -v mydb < large_file.sql

# 或使用 pv 监控进度
pv large_file.sql | mysql -u root -p mydb
```

### 调整 MySQL 配置

如果导入大文件时遇到问题，可以调整以下配置：

```sql
-- 查看当前配置
SHOW VARIABLES LIKE 'max_allowed_packet';
SHOW VARIABLES LIKE 'net_buffer_length';

-- 临时增加允许的包大小（单位：字节）
SET GLOBAL max_allowed_packet = 268435456;  -- 256MB

-- 增加网络缓冲区
SET GLOBAL net_buffer_length = 1048576;    -- 1MB
```

### 注意事项

- 修改配置后需要重新连接数据库才能生效
- 生产环境修改配置前请评估影响

## 0x06 验证导入结果

导入完成后，可以验证数据是否正确导入：

```sql
-- 查看所有表
SHOW TABLES;

-- 查看表结构
DESC users;
DESC products;

-- 查看数据
SELECT * FROM users;
SELECT * FROM products;

-- 统计行数
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM products;
```

## 0x07 常见问题处理

### 中文乱码

如果导入后出现中文乱码，检查以下几点：

1. 确保 SQL 文件编码为 UTF-8
2. 创建数据库时指定正确的字符集
3. 导入时指定字符集

```sql
-- 创建数据库时指定字符集
CREATE DATABASE mydb CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 导入时指定字符集
mysql -u root -p --default-character-set=utf8mb4 mydb < init.sql
```

### 导入中断

如果导入过程中断，可能的原因：

1. SQL 语法错误
2. 外键约束失败
3. 唯一键冲突

解决方法：

```sql
-- 暂时禁用外键检查
SET FOREIGN_KEY_CHECKS = 0;

-- 执行导入...

-- 重新启用外键检查
SET FOREIGN_KEY_CHECKS = 1;
```

### 权限问题

如果遇到权限错误：

```sql
-- 授予用户权限
GRANT ALL PRIVILEGES ON mydb.* TO 'username'@'localhost';
FLUSH PRIVILEGES;
```

## 参考

- [MySQL 8.0 Reference Manual - Database Administration](https://dev.mysql.com/doc/refman/8.0/en/database-administration.html)
- [MySQL 8.0 - mysql Client](https://dev.mysql.com/doc/refman/8.0/en/mysql.html)
- [MySQL 8.0 - SHOW TABLES](https://dev.mysql.com/doc/refman/8.0/en/show-tables.html)

---

# PostgreSQL 13 导入外部SQL文件

> 本节介绍在 PostgreSQL 13 中导入外部 SQL 文件的常用方法。

## 0x01 前置准备

在导入 SQL 文件之前，需要确保：

- PostgreSQL 13 服务已启动
- 已创建目标数据库（如果 SQL 文件中不包含创建数据库的语句）
- 拥有数据库的操作权限

```bash
# 登录 PostgreSQL
psql -U postgres

# 创建目标数据库（如果需要）
CREATE DATABASE mydb;
```

## 0x02 使用 psql 命令行导入

`psql` 是 PostgreSQL 客户端的命令行工具，适合导入各种大小的 SQL 文件。

### 基本语法

```bash
psql -U username -d database_name -f /path/to/yourfile.sql
```

### 参数说明

| 参数 | 说明 |
|------|------|
| `-U` | 指定用户名 |
| `-d` | 目标数据库名 |
| `-f` | 指定要执行的 SQL 文件 |
| `-h` | 主机地址（默认 localhost） |
| `-p` | 端口（默认 5432） |

### 示例

```bash
# 导入到指定数据库
psql -U postgres -d mydb -f init.sql

# 指定主机和端口
psql -U postgres -h localhost -p 5432 -d mydb -f init.sql

# 如果 SQL 文件包含创建数据库语句
psql -U postgres -f full_backup.sql

# 静默模式（不输出执行信息）
psql -U postgres -d mydb -f init.sql -q
```

## 0x03 使用 psql 交互式导入

在 psql 客户端内执行 `\i` 命令。

### 基本语法

```sql
\i /path/to/yourfile.sql
```

### 示例

```bash
# 进入 psql 客户端
psql -U postgres -d mydb

-- 在 psql 内执行
mdb=> \i /home/user/data/init.sql

-- Windows 路径示例
mdb=> \i C:\Users\admin\data\init.sql
```

### 注意事项

- 文件路径可以是绝对路径或相对路径
- 如果 SQL 文件包含连接其他数据库的语句，确保目标数据库存在

## 0x04 导入包含建表和数据的 SQL 文件

下面是一个完整的示例 SQL 文件，包含了创建表和插入数据：

```sql
-- init_database.sql

-- 创建扩展（如果需要）
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- 创建用户表
CREATE TABLE IF NOT EXISTS users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 创建商品表
CREATE TABLE IF NOT EXISTS products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock INT DEFAULT 0,
    category_id INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 插入用户数据
INSERT INTO users (username, email, password_hash) VALUES
('admin', 'admin@example.com', '$2b$10$abcdefghijklmnopqrstuv'),
('user1', 'user1@example.com', '$2b$10$zyxwvutsrqponmlkjihgf'),
('user2', 'user2@example.com', '$2b$10$ABCDEFGHIJKLMNOPQRSTU');

-- 插入商品数据
INSERT INTO products (name, description, price, stock, category_id) VALUES
('笔记本电脑', '15.6英寸轻薄本', 5999.00, 50, 1),
('无线鼠标', '蓝牙5.0无线鼠标', 89.00, 200, 2),
('机械键盘', '青轴机械键盘', 299.00, 80, 2);
```

### 导入示例

```bash
# 方法一：使用 psql 命令行
psql -U postgres -d shop -f init_database.sql

# 方法二：在 psql 客户端内
psql -U postgres -d shop
shop=> \i /path/to/init_database.sql
```

## 0x05 导入大文件的注意事项

### 使用 psql 导入

对于大文件，建议使用 psql 命令行：

```bash
# 启用详细输出
psql -U postgres -d mydb -f large_file.sql -v ON_ERROR_STOP=1

# 使用 pv 监控进度
pv large_file.sql | psql -U postgres -d mydb
```

### 调整 PostgreSQL 配置

如果导入大文件时遇到问题，可以调整以下配置：

```sql
-- 查看当前配置
SHOW max_connections;
SHOW shared_buffers;
SHOW work_mem;

-- 临时增加工作内存
SET work_mem = '256MB';
```

### 注意事项

- PostgreSQL 默认单条 SQL 语句大小限制较小
- 大文件建议分批次导入或使用 `pg_dump` / `pg_restore`

## 0x06 验证导入结果

导入完成后，可以验证数据是否正确导入：

```sql
-- 查看所有表
\dt

-- 查看表结构
\d users
\d products

-- 查看数据
SELECT * FROM users;
SELECT * FROM products;

-- 统计行数
SELECT COUNT(*) FROM users;
SELECT COUNT(*) FROM products;
```

## 0x07 常见问题处理

### 中文乱码

如果导入后出现中文乱码，检查以下几点：

1. 确保 SQL 文件编码为 UTF-8
2. 创建数据库时指定正确的编码

```bash
# 创建数据库时指定编码
psql -U postgres
postgres=> CREATE DATABASE mydb WITH ENCODING 'UTF8';
```

### 导入中断

如果导入过程中断，可能的原因：

1. SQL 语法错误
2. 外键约束失败
3. 唯一键冲突

解决方法：

```sql
-- 暂时禁用外键约束
SET CONSTRAINTS ALL DEFERRED;

-- 执行导入...

-- 重新启用外键约束（如果有问题会自动报错）
```

### 权限问题

如果遇到权限错误：

```sql
-- 授予用户权限
GRANT ALL PRIVILEGES ON DATABASE mydb TO username;

-- 授予表权限
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO username;
GRANT ALL PRIVILEGES ON ALL SEQUENCES IN SCHEMA public TO username;
```

## 参考

- [PostgreSQL Documentation - psql](https://www.postgresql.org/docs/13/app-psql.html)
- [PostgreSQL Documentation - SQL Syntax](https://www.postgresql.org/docs/13/sql.html)