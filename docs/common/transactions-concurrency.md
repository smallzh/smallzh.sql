# 事务与并发

## 0x01 事务基础

### 什么是事务？

> 事务是数据库操作的逻辑单位，它将一系列操作作为一个整体执行，要么全部成功，要么全部失败。

### ACID 特性

| 特性 | 说明 | 示例 |
|------|------|------|
| **原子性（Atomicity）** | 事务中的操作要么全部成功，要么全部失败 | 转账：扣款和加款必须同时成功或失败 |
| **一致性（Consistency）** | 事务前后数据库保持一致状态 | 转账前后总金额不变 |
| **隔离性（Isolation）** | 并发事务之间相互隔离 | 两个转账事务互不干扰 |
| **持久性（Durability）** | 事务提交后，数据永久保存 | 提交后的数据不会丢失 |

## 0x02 事务操作

### 基本事务控制

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

### 自动提交模式

```sql
-- MySQL: 查看自动提交状态
SHOW VARIABLES LIKE 'autocommit';

-- 关闭自动提交
SET autocommit = 0;

-- 开启自动提交
SET autocommit = 1;

-- PostgreSQL: 查看自动提交状态
SHOW autocommit;
```

### 保存点（Savepoint）

```sql
BEGIN;

-- 第一个操作
UPDATE accounts SET balance = balance - 100 WHERE id = 1;

-- 创建保存点
SAVEPOINT point1;

-- 第二个操作
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- 回滚到保存点（撤销第二个操作，保留第一个）
ROLLBACK TO point1;

-- 继续其他操作
UPDATE accounts SET balance = balance + 100 WHERE id = 3;

-- 提交事务
COMMIT;
```

## 0x03 并发问题

### 脏读（Dirty Read）

读取到其他事务未提交的数据：

```sql
-- 事务 A
BEGIN;
UPDATE accounts SET balance = 1000 WHERE id = 1;
-- 未提交

-- 事务 B（如果隔离级别允许脏读）
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 可能读到 1000
COMMIT;

-- 事务 A 回滚
ROLLBACK;  -- 事务 B 读到的数据是无效的
```

### 不可重复读（Non-repeatable Read）

同一事务内，两次读取同一数据得到不同结果：

```sql
-- 事务 A
BEGIN;
SELECT balance FROM accounts WHERE id = 1;  -- 读到 500
-- 事务 B 在此时修改并提交
SELECT balance FROM accounts WHERE id = 1;  -- 可能读到 600
COMMIT;
```

### 幻读（Phantom Read）

同一事务内，两次查询得到不同数量的行：

```sql
-- 事务 A
BEGIN;
SELECT COUNT(*) FROM accounts WHERE balance > 1000;  -- 结果: 5
-- 事务 B 在此时插入新记录并提交
SELECT COUNT(*) FROM accounts WHERE balance > 1000;  -- 可能结果: 6
COMMIT;
```

## 0x04 隔离级别

### 隔离级别对比

| 隔离级别 | 脏读 | 不可重复读 | 幻读 | 说明 |
|----------|------|------------|------|------|
| READ UNCOMMITTED | ✓ | ✓ | ✓ | 最低隔离，性能最好 |
| READ COMMITTED | ✗ | ✓ | ✓ | 多数数据库默认 |
| REPEATABLE READ | ✗ | ✗ | ✓ | MySQL 默认 |
| SERIALIZABLE | ✗ | ✗ | ✗ | 最高隔离，性能最差 |

### 设置隔离级别

```sql
-- MySQL: 查看当前隔离级别
SELECT @@transaction_isolation;

-- MySQL: 设置会话隔离级别
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- MySQL: 设置全局隔离级别
SET GLOBAL TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- PostgreSQL: 查看当前隔离级别
SHOW transaction_isolation;

-- PostgreSQL: 设置事务隔离级别（必须在事务开始时设置）
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### 各隔离级别示例

```sql
-- READ COMMITTED 示例
BEGIN;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT balance FROM accounts WHERE id = 1;  -- 500
-- 其他事务修改并提交
SELECT balance FROM accounts WHERE id = 1;  -- 可能 600（不可重复读）
COMMIT;

-- REPEATABLE READ 示例
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT balance FROM accounts WHERE id = 1;  -- 500
-- 其他事务修改并提交
SELECT balance FROM accounts WHERE id = 1;  -- 仍然是 500（可重复读）
COMMIT;

-- SERIALIZABLE 示例
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT COUNT(*) FROM accounts WHERE balance > 1000;  -- 5
-- 其他事务尝试插入会等待或失败
SELECT COUNT(*) FROM accounts WHERE balance > 1000;  -- 仍然是 5
COMMIT;
```

## 0x05 锁机制

### 锁的类型

#### 共享锁（Shared Lock / S Lock）

```sql
-- MySQL: 共享锁
BEGIN;
SELECT * FROM accounts WHERE id = 1 LOCK IN SHARE MODE;
-- 其他事务可以读取，但不能修改
COMMIT;

-- PostgreSQL: 共享锁
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR SHARE;
COMMIT;
```

#### 排他锁（Exclusive Lock / X Lock）

```sql
-- MySQL: 排他锁
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- 其他事务不能读取（除非使用非锁定读）或修改
COMMIT;

-- PostgreSQL: 排他锁
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
COMMIT;
```

### 悲观锁与乐观锁

#### 悲观锁

```sql
-- 使用 SELECT ... FOR UPDATE 实现悲观锁
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
```

#### 乐观锁

```sql
-- 使用版本号实现乐观锁
CREATE TABLE products (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    stock INT,
    version INT DEFAULT 1
);

-- 更新时检查版本号
UPDATE products 
SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 1;  -- 如果版本号不匹配，更新失败
```

### 死锁处理

```sql
-- 死锁示例
-- 事务 A
BEGIN;
UPDATE accounts SET balance = 100 WHERE id = 1;  -- 锁定 id=1
UPDATE accounts SET balance = 200 WHERE id = 2;  -- 等待事务 B 释放 id=2
COMMIT;

-- 事务 B
BEGIN;
UPDATE accounts SET balance = 200 WHERE id = 2;  -- 锁定 id=2
UPDATE accounts SET balance = 100 WHERE id = 1;  -- 等待事务 A 释放 id=1（死锁）
COMMIT;

-- 解决方案：按固定顺序访问资源
-- 事务 A 和 B 都先访问 id=1，再访问 id=2
```

## 0x06 MVCC 多版本并发控制

### MVCC 原理

> MVCC（Multi-Version Concurrency Control）通过保存数据的多个版本来实现并发控制，读操作不阻塞写操作，写操作不阻塞读操作。

### PostgreSQL MVCC

```sql
-- 查看事务 ID
SELECT txid_current();

-- 查看行版本信息
SELECT xmin, xmax, * FROM accounts WHERE id = 1;

-- 创建快照
BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SELECT pg_export_snapshot();  -- 导出快照
```

### MySQL InnoDB MVCC

```sql
-- InnoDB 使用 undo log 实现 MVCC
-- 查看当前事务 ID
SELECT trx_id FROM information_schema.innodb_trx 
WHERE trx_mysql_thread_id = CONNECTION_ID();
```

## 0x07 分布式事务

### 两阶段提交（2PC）

```sql
-- MySQL XA 事务
-- 阶段 1: 准备
XA START 'tx1';
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
XA END 'tx1';
XA PREPARE 'tx1';

-- 阶段 2: 提交/回滚
XA COMMIT 'tx1';  -- 或 XA ROLLBACK 'tx1'
```

### PostgreSQL 两阶段提交

```sql
-- 准备事务
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
PREPARE TRANSACTION 'tx1';

-- 提交/回滚
COMMIT PREPARED 'tx1';  -- 或 ROLLBACK PREPARED 'tx1'
```

## 0x08 事务最佳实践

### 事务设计原则

```sql
-- 1. 保持事务简短
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;  -- 快速提交

-- 2. 避免长事务
-- 不好：事务中包含大量操作或用户交互
BEGIN;
-- 大量操作...
-- 等待用户输入...
COMMIT;

-- 3. 合理使用隔离级别
-- 大多数应用使用 READ COMMITTED 即可
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```

### 错误处理

```sql
-- MySQL 存储过程中的事务处理
DELIMITER //
CREATE PROCEDURE transfer_money(IN from_id INT, IN to_id INT, IN amount DECIMAL(10,2))
BEGIN
    DECLARE EXIT HANDLER FOR SQLEXCEPTION
    BEGIN
        ROLLBACK;
        SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Transfer failed';
    END;
    
    START TRANSACTION;
    UPDATE accounts SET balance = balance - amount WHERE id = from_id;
    UPDATE accounts SET balance = balance + amount WHERE id = to_id;
    COMMIT;
END //
DELIMITER ;

-- PostgreSQL 错误处理
DO $$
BEGIN
    BEGIN
        UPDATE accounts SET balance = balance - 100 WHERE id = 1;
        UPDATE accounts SET balance = balance + 100 WHERE id = 2;
    EXCEPTION WHEN OTHERS THEN
        RAISE NOTICE 'Error occurred: %', SQLERRM;
        -- 自动回滚
    END;
END $$;
```

## 参考

- [MySQL Transactions](https://dev.mysql.com/doc/refman/8.0/en/commit.html)
- [PostgreSQL Transactions](https://www.postgresql.org/docs/13/tutorial-transactions.html)
- [MySQL Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-transaction-model.html)
- [PostgreSQL Locking](https://www.postgresql.org/docs/13/explicit-locking.html)
- [MySQL Isolation Levels](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html)
- [PostgreSQL Isolation Levels](https://www.postgresql.org/docs/13/transaction-iso.html)