---
title: "循序渐进 MySQL 事务隔离级别"
date: 2016-12-10T18:14:45+08:00
draft: false
comment: true
tags: ["mysql"]
---

本篇文章的重点在于总结MYSQL事务。

# 什么是事务

事务简言之就是一组 SQL 执行要么全部成功，要么全部失败。MYSQL 的事务在存储引擎层实现。

事务都有 ACID 特性：

- 原子性（Atomicity）：一个事务必须被视为一个不可分割的单元；
- 一致性（Consistency）：数据库总是从一种状态切换到另一种状态；
- 隔离性（Isolation）：通常来说，事务在提交前对于其他事务不可见；
- 持久性（Durablity）：一旦事务提交，所做修改永久保存数据库；

事务最常用的例子就是银行转账。假设 polo 需给 lynn 转账1000元，如下步骤：

- 确认 polo 账户余额高于1000元；
- 从 polo 的账户余额减去1000元；
- 将 lynn 的账户余额增加1000元；

SQL语句如下:

```sql
mysql> BEGIN;
mysql> SELECT balance FROM bank_account WHERE uid=10001;
mysql> UPDATE bank_account SET balance=balance-1000 WHERE uid=10001;
mysql> UPDATE bank_account SET balance=balance+1000 WHERE uid=10002;
mysql> COMMIT;
```

mysql 启动事务可使用 BEGIN 或 START TRANSACTION；上述三个步骤执行在一个事务中就能够保证数据的完整性，要么全部成功，要么全部失败。

MYSQL 提供两种事务型引擎：Innodb 和 NDBCluster。默认采用自动提交模式，执行一条语句自动 COMMIT。通过 AUTOCOMMIT 变量可启用或者禁用自动提交模式：

```sql
mysql> SHOW VARIABLES LIKE "AUTOCOMMIT";
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| autocommit    | ON    |
+---------------+-------+
1 row in set (0.00 sec)

mysql> SET AUTOCOMMIT=1
```

AUTOCOMMIT=1 表示开启默认提交，0表示关闭默认提交需要手动提交。

# 隔离级别

事务隔离性的解释：通常情况下，事务在提交之前对于其他事务不可见。

数据库有四种隔离级别，当然 MYSQL 也是如此。分别是：

- 未提交读，READ UNCOMMITTED
- 已提交读，READ COMMITTED
- 可重复读，REPEATABLE READ
- 串行化，SEAIALIZABLE

关于隔离级别的两个理解

- 书本解释，每种级别都规定了一个事务中所做修改，哪些在事务内和事务间是可见的。
- 我的理解，隔离级别就是决定一个事务的修改另一个事务什么情况下能看到。

两者区别在于是否存在事务内可见性，但无论哪个级别在事务内的操作肯定是可见的，重点在事务间可见性。

下面开始说明 MYSQL 的四种隔离级别，先准备一张学生表：

```sql
mysql> CREATE TABLE `student` (
 `id` int(11) NOT NULL AUTO_INCREMENT,
 `name` varchar(32) NOT NULL DEFAULT '',
 PRIMARY KEY (`id`)
) DEFAULT CHARSET=utf8 |
```

只有id（主键自增）与name字段

# 未提交读

事务中修改没有提交对其他事务也是可见的，俗称脏读。非常不建议使用。

示例演示,客户端A和B设置隔离级别为未提交读
```sql
mysql> SET SESSION TX_ISOLATION='READ-UNCOMMITTED';
```
客户端A与B开启事务并查询student

```sql
mysql> BEGIN； mysql> SELECT * FROM student;
Empty set (0.00 sec)
```

当前，客户端 A 和 B 都是空数据。此时在客服端 B 插入一条新的数据

```sql
mysql> INSERT INTO student(name) VALUES("polo");
Query OK, 1 row affected (0.00 sec)
```

此时事务还未提交，客服端 A 查看一下 student 表，如下：
```sql
mysql> SELECT * FROM student;
+----+------+
| id | name |
+----+------+
|  1 | polo |
+----+------+
1 row in set (0.00 sec)
```

可以看出，客户端 A 看到 B 未提交的修改。客户端 B 执行回滚操作，如下：

```sql
mysql> ROLLBACK
```

成功之后，客户端 A 查看 student 表：

```sql
mysql> SELECT * FROM student;
Empty set (0.00 sec)
```

输出显示，客户端A查看数据为空。

以上可以看出未提交读隔离级别的危险性，对于一个没有提交事务所做修改对另一个事务是可见状态，容易造成脏读。非特殊情况不得使用此级别

# 读提交读

多数数据库系统默认为此级别（MYSQL不是）。已提交读级别即为一个事务只能看到已提交事务所做的修改，也就解决了未提交读的问题，即脏读的问题。

示例演示，客户端A和B设置隔离级别为已提交读，执行如下命令：

```sql
mysql> SET SESSION TX_ISOLATION='READ-COMMITTED';
```

客户端 A与 B 开启事务并查询 student

```sql
mysql> BEGIN;
mysql> SELECT * FROM student;
Empty set (0.00 sec)
```

结果显示，客户端A和B都为空。接着，客户端 B 插入一条新的数据但不执行提交：

```sql
mysql> INSERT INTO student (name) VALUES('polo');
```

接下来，客户端 A 查看一下 student 数据：

```sql
mysql> SELECT * FROM student;
Empty set (0.00 sec)
```

注意这里与上面不同了，在客户端B没有提交事务情况下无数据。下面客户端B提交事务：

```sql
mysql> COMMIT;
```

客户端 A 再查看下 student 表。

```sql
mysql> SELECT * FROM student;
+----+------+
| id | name |
+----+------+
|  1 | polo |
+----+------+
1 row in set (0.00 sec)
```

这样我们就成功读取到了客户。

从上面的示例可以看出，提交读没有了脏读问题，但我们可以看到在客户端 A 的一个事务中执行两次同样的 SELECT 语句得到不同结果，因此已提交读又被称为不可重复读。同样筛选条件可能得到不同的结果。

# 可重复读

如其名所言，解决已提交读不可重复读取的问题。

示例演示，客户端A和B设置隔离级别为可重复读。首先设置隔离级别：

```sql
mysql> SET SESSION tx_isolation='REPEATABLE-READ'
```

客户端A与B开启事务并查看

```sql
mysql> BEGIN;
mysql> SELECT * FROM student;
+----+------+
| id | name |
+----+------+
|  1 | polo |
+----+------+
1 rows in set (0.00 sec)
```

客服端 B 更新 polo 为 adam 并提交事务

```sql
mysql> UPDATE student SET name='adam' WHERE id=1;
mysql> COMMIT
```

客户端A查看student表，结果如下：

```sql
mysql> SELECT * FROM student;
+----+------+
| id | name |
+----+------+
|  1 | polo |
+----+------+
1 rows in set (0.00 sec)
```

客户端 A 查看数据未变，没有不可重复读问题。

客户端 A 提交事务，并查看student表。

```sql
mysql> COMMIT; mysql> SELECT * FROM student;
+----+------+
| id | name |
+----+------+
|  1 | polo |
+----+------+
1 rows in set (0.00 sec)
```

从上面的示例可知，可重复读两次读取内容一样。该级别并没有解决幻读的问题。但是MYSQL在可重复读基础上增加了MVCC机制解决了此问题，此处无法演示幻读的效果。

那什么是幻读？首先，可重复读锁定范围为当前查询到的内容，如执行

```sql
mysql> SELECT * FROM student WHERE id>=1
```

锁定的即 id>=1 查到的行，为行级锁。如另一事务执行并默认提交以下语句

```sql
mysql> INSERT INTO student (name) VALUES ('stephen');
```

新增的这行并没有被锁定，此时读取 student

```sql
mysql> SELECT * FROM student WHERE id>=1;
+----+---------+
| id | name    |
+----+---------+
|  1 | polo    |
|  2 | stephen |
+----+---------+
2 rows in set (0.00 sec)
```

出现了幻读。关于这个问题，除了使用 MYSQL 的 MVCC 机制，还可以用可串行化隔离级别解决此问题。

# 串行化

串行化是最高隔离级别，强制事务串行执行。执行串行了，那么也就解决了一切的问题，这个级别只有在对数据一致性要求非常严格且没用并发的情况下使用。

示例演示，客户端 A 和 B 设置隔离级别为可串行化。
```sql
mysql> SET SESSION tx_isolation='SERIALIZABLE';
```

首先，客户端 A 执行一下查询：
```sql
mysql> SELECT * FROM student WHERE id<4;
+----+---------+
| id | name    |
+----+---------+
|  1 | polo    |
|  2 | stephen |
+----+---------+
2 rows in set (0.00 sec)
```

接下来，客户端 B 执行数据新增：

mysql> INSERT INTO student (name) VALUES('yunteng');
好的！效果出现了，此时我们会发现INSERT语句被阻塞执行，原因就是A执行了查询表student同时满足`id<4`，已被锁定。如果查询表student条件为`id<3`，则新增语句可正常执行。

# 汇总图

隔离级别 |  英文  |  脏读      | 不可重复读  | 幻读    | 加锁读
-------- | ---------- | ---------- | ------ | ------ |-------
未提交读 | READ UNCOMMITTED | 是         | 是          | 是      | 否
提交读  | READ COMMITTED | 否         | 是          | 是     | 否
可重复读 | REPEATABLE READ | 否         | 否          | 是      | 否
串行化  | SERIALIZABLE | 否          | 否          | 否      | 是

关于事务的隔离级别到此结束。具体使用何种情况还要根据具体的业务场景来决定。

