---
layout: post
title: 事务隔离级
categories: db
tags : transaction acid
---

### 概念及其之间的关系  

文章开头先理下这些个概念的关系

1.  并发环境下，我们要求我们的事务必须具备`ACID`特性。
2.  对于1的I（isolation）隔离性，我们制定了四个隔离级(`RU`、`RC`、`RR`、`S`)，根据不同的业务场景可进行调整（一般就是RC，RR）
3.  不同的隔离级下会产生一些不好的读现象（`脏读`、`不可重复读`、`幻读`）。
4.  针对不同的sql，mysql会采用不同的读操作（`快照读`、`当前读`）
5.  快照读不加锁（也有例外，比如隔离级为S时,具体[参见](http://mysql.taobao.org/monthly/2016/01/01/)）,而对于当前读，会有不同的加锁策略(`记录锁`、`间隙锁`、`next－key锁`)，某一加锁策略下，可能还会对应不同的加锁类型(`X锁`、`S锁`)
6.  spring中针对某一隔离级还会有不同的传播特性。

然后接着看下文吧。

#### acid

[InnoDB and the ACID Model](https://dev.mysql.com/doc/refman/5.7/en/mysql-acid.html)

1.  A 原子性　atomic 事务要么全部完成，要么全部取消，即使它持续运行10个小时。如果事务崩溃，状态回到事务之前（事务回滚）
2.  C 一致性  consistency 只有合法的数据（依照关系约束和函数约束）能写入数据库，一致性与原子性和隔离性有关
3.  I 隔离性  isolation 事务的执行不受其他并发事务的的影响。
4.  D 持久性  durability 一旦事务提交（也就是成功执行）,不管发生什么（崩溃或者出错），数据要保存在数据库中。

#### 事务隔离级

1.  读未提交 read uncommitted
2.  读提交  read committed
3.  可重复读 repeatable read
4.  串行  serializable

#### 读现象 

1.  脏读 dirty_read 读到别人未提交的数据。隔离级为RU时，会出现
2.  不可重复读 Non-repeatable reads  同一事务中两次读的记录不同。隔离级为RU,RC时会出现
3.  幻读 Phantom reads 同一事务中，进行count操作，两次结果不一样。隔离级为RU,RC,RR时会出现 [Phantom_reads](https://en.wikipedia.org/wiki/Isolation_%28database_systems%29%23Phantom_reads)

ps: 针对幻读,mysql通过mvcc使得在RR级别下不会出现幻读现象，注意下。

参见

[(Isolation (database systems))](https://en.wikipedia.org/wiki/Isolation_%28database_systems%29)    

[MySQL 加锁处理分析](http://hedengcheng.com/?p=771)

#### 锁类型

数据库中的基本锁机制：排它锁（写锁，X锁）和共享锁（读锁，S锁） [InnoDB Locking](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html)

**排他锁** exclusive (X) lock 事务T对数据对象A加了X锁，其他事务就不能对A加锁了，直到T释放A上的锁，这样就保证了其他事务在T释放A上的锁之前对A加X锁或S锁   
**共享锁** shared (S) lock事务T对数据对象A加了S锁，可以对读A但是不能写A，其他事务可以对A加S锁，但是不能加X锁，直到事务T释放A上的锁。
**意向排他锁**
**意向共享锁**

#### 锁算法

* 记录锁 Record Locks  Record locks always lock index records（这里的索引包含聚集索引和二级索引，都会加锁）
* 间隙锁 Gap Locks  A gap lock is a lock on a gap between index records, or a lock on the gap before the first or after the last index record
* next-key锁

#### innodb的两类读操作

1.  快照读 (snapshot read、consistent read) 读取的是记录的可见版本 (有可能是历史版本)，不用加锁。
    1.  简单的select操作，属于快照读，不加锁(隔离级为serializable除外)
    2.  select * from table where ?;
2.  当前读 (current read、locking read) 读取的是记录的最新版本，并且，当前读返回的记录，都会加上锁，保证其他事务不会再并发修改这条记录。
    1.  特殊的读操作，插入/更新/删除操作，属于当前读，需要加锁。
    2.  lock in share mode
    3.  for update
    4.  insert
    5.  delete
    6.  update

#### 并发控制协议
1.  多版本并发控制 MVCC（Multi-Version Concurrency Contro） 读不加锁，读写不冲突。在读多写少的OLTP应用中，读写不冲突是非常重要的，极大的增加了系统的并发性能，这也是为什么现阶段，几乎所有的RDBMS，都支持了MVCC
2.  基于锁的并发控制 (Lock-Based Concurrency Control)

结束事务的两种不同的方式：回滚或者提交

#### two-parse locking

2PL就是将加锁/解锁分为两个完全不相交的阶段。加锁阶段：只加锁，不放锁。解锁阶段：只放锁，不加锁。

![2pl](/images/database/2pl.jpg)

### 参考

[SET TRANSACTION Syntax]<http://dev.mysql.com/doc/refman/5.7/en/set-transaction.html#isolevel_read-committed>

[Consistent NonLocking Reads]<http://dev.mysql.com/doc/refman/5.7/en/innodb-consistent-read.html>

[locking read]<http://dev.mysql.com/doc/refman/5.7/en/glossary.html#glos_locking_read>

[mysql 加锁处理分析]<http://hedengcheng.com/?p=771>

[MySQL · 引擎特性 · InnoDB 事务锁系统简介]<http://mysql.taobao.org/monthly/2016/01/01/>
