---
title: mysql异常排查sql
date: 2025-06-05 00:09:57
categories: mysql
tag: 备份
---

###查找持续时间超过 60s 的事务
```sql
select * from information_schema.innodb_trx where TIME_TO_SEC(timediff(now(),trx_started))>60
```
###指定sql的引擎
```sql
create table test(c int) engine=InnoDB;
```
###查看 transaction-isolation 的值,例如： READ-COMMITTED
```sql
show variables like 'transaction_isolation';
```
### alter table 语句里面设定等待时间，如果在这个指定的等待时间里面能够拿到 MDL 写锁最好，拿不到也不要阻塞后面的业务语句，先放弃。之后开发人员或者 DBA 再通过重试命令重复这个过程。
```sql
alter table t nowait add column c int(11) not null default 0;
alter table t wait add column c int(11) not null default 0;

```
### 防止出现死锁，设置加锁超时时间
```sql
SHOW GLOBAL VARIABLES LIKE 'innodb_lock_wait_timeout';
SET GLOBAL innodb_lock_wait_timeout=120;
```
### 发起死锁检测，主动回滚死锁链条中的某一个事务，让其他事务得以继续执行。将参数 innodb_deadlock_detect 设置为 on，表示开启这个逻辑。
```sql
SHOW GLOBAL VARIABLES LIKE 'innodb_deadlock_detect';
SET GLOBAL innodb_deadlock_detect=on;
```


# MySQL 中 SELECT 与 ALTER TABLE 操作对表锁定的影响比较
## 一、SELECT 查询的表锁定行为
| 存储引擎       | 锁定行为                                          |
| ---------- | --------------------------------------------- |
| **InnoDB** | 默认使用 **非锁定一致性读**（MVCC 实现），**不锁表**。            |
| **MyISAM** | 获取**共享锁（读锁）**，为**表级锁**，会**阻塞写操作**，但不会阻塞其他读操作。 |
## 二、ALTER TABLE 操作的锁定行为
### 1. InnoDB 引擎
| MySQL 版本    | 操作类型           | 锁定行为                                           |
| ----------- | -------------- | ---------------------------------------------- |
| MySQL 5.6+  | 在线 DDL（如添加列）   | 支持使用 `ALGORITHM=INPLACE` 和 `LOCK=NONE`，**不锁表** |
| MySQL 5.6+  | 复制数据操作（如修改列类型） | 使用 `ALGORITHM=COPY`，**会锁表**                    |
| MySQL ≤ 5.5 | 所有 ALTER 操作    | **大多数会锁表**，阻塞读写操作                              |
### 2. MyISAM 引擎
* 所有 ALTER TABLE 操作都会**锁定整个表**，阻塞所有读写操作直到完成。
## 三、减少锁定影响的策略
* ✅ 使用在线 DDL（MySQL 5.6+）：
  ```sql
  ALTER TABLE table_name ADD COLUMN new_column INT, ALGORITHM=INPLACE, LOCK=NONE;
  ```
* ✅ 在业务低峰期执行 ALTER TABLE。
* ✅ 使用工具 **pt-online-schema-change**（Percona Toolkit）对大表结构进行无锁修改。
* ✅ 主从复制架构下，先在从库执行 ALTER TABLE，验证无误后切主。
## 四、示例对比
```sql
-- 在线添加列（不锁表）
ALTER TABLE users ADD COLUMN phone VARCHAR(20), ALGORITHM=INPLACE, LOCK=NONE;
-- 修改列类型（可能锁表）
ALTER TABLE users MODIFY COLUMN phone VARCHAR(30), ALGORITHM=COPY;
```
## 五、总结对比
| 操作类型        | InnoDB（默认）       | MyISAM      |
| ----------- | ---------------- | ----------- |
| SELECT      | 非锁定一致性读，不锁表      | 表级共享锁，阻塞写操作 |
| ALTER TABLE | 支持在线 DDL，部分操作不锁表 | 所有操作锁表，阻塞读写 |
