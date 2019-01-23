---
layout: post
title: sql基础
date: 2014-11-16
categories: db
tags:
    - sql
    - 范式
---

1.  ps

    1.  update product_details set weight=38, exist = 1 where name = 'jim'// 更新操作,and是错误的　

    2.  create table select ：创建一个和原表字段结构一致的新表，去掉所有的约束，同时将原表select的结果数据插入新表　　　　

    3.  create table like ： 创建一个和原表结构完全一致的新空表，包含全部约束　　　　

    4.  on 后连接的字段要考虑设计索引。　　

    5.  show variables like 'log_slow_queries';　查看是否启用慢日志　

    6.  show  variables like 'long_query_time';　查看慢于多少秒的sql会记录到慢日志中

    7.  涉及数据库的字段更改,需要考虑读取,写入的时间段,考虑 ***兼容性***,  ***老数据***,`发布过程服务不能停`,注意***版本***的概念.

    8.   当查询的列不是独立的，而是`表达式`或者`函数`的一部分时，这种情况下，mysql将无法使用该列的索引,所以合理的做法是在查之前,就计算出该值

    9.   当查询的列需要进行`模糊匹配`时，即使该列建了索引，也需要进行全表扫描

    10.   最早的mysql一次查询只能够使用其中一个索引，5.0即以后，mysql引入了索引合并（index merge）的策略，但是完全可以通过建立多列索引来避免索引合并给数据库带来额外的开销,以达到更好的性能。

## 范式

### 基本概念

`部分函数依赖`：
设X,Y是关系R的两个属性集合，存在X→Y，若X’是X的真子集，存在X’→Y，则称Y部分函数依赖于X。

举个例子：学生基本信息表R中（学号，身份证号，姓名）当然学号属性取值是唯一的，在R关系中，（学号，身份证号）->（姓名），（学号）->（姓名），（身份证号）->（姓名）；所以姓名部分函数依赖与（学号，身份证号）；

`完全函数依赖`：
设X,Y是关系R的两个属性集合，X’是X的真子集，存在X→Y，但对每一个X’都有X’!→Y，则称Y完全函数依赖于X。

例子：学生基本信息表R（学号，班级，姓名）假设不同的班级学号有相同的，班级内学号不能相同，在R关系中，（学号，班级）->（姓名），但是（学号）->(姓名)不成立，（班级）->(姓名)不成立，所以姓名完全函数依赖与（学号，班级）；

`传递函数依赖`：
设X,Y,Z是关系R中互不相同的属性集合，存在X→Y(Y !→X),Y→Z，则称Z传递函数依赖于X。

例子：在关系R(学号 ,宿舍, 费用)中，(学号)->(宿舍),宿舍！=学号，(宿舍)->(费用),费用!=宿舍，所以符合传递函数的要求；

### 一范式

### 二范式

### 三范式

### BC范式

## 索引

1.  查看表的索引　　

        show index from tb_xx;
2.  添加表的索引,唯一索引，主键　　

        alter table tb_xx add index `idx_site_channel_status`(`site`,`channel`,`status`)
        alter table tb_xx unique `uniq_name`(`name`)
        alter table tb_xx primary key  `id` (`id`)
3.  删除表的索引　　

        alter table tb_xx drop index `idx_site_channel_status`　　　/　　drop index `idx_site_channel_status` on tb_xx;
        alter table tb_xx drop index `uniq_name` (*跟删除普通的索引是一样的*)
        alter table tb_xx drop primary key;
    ps: **auto_increment只能作用于key,删除有auto_increment约束的索引时，必须先将auto_increment约束删除**

4.  尽量不要在索引列上进行运算，使用函数，这样就用不到索引了。原因很简单b+树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大
5.  不建议使用过长的字段作为主键，因为所有辅助索引都引用主索引，过长的主索引会令辅助索引变得过大 参看[mysql执行计划#索引的数据结构](/2016/07/16/mysql_explain#ds)
6.  尽量选择区分度高的列作为索引,区分度的公式是count(distinct col)/count(\*)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1

## other

COLLATE=utf8_bin

    select 'a' = 'A';
    1
    select binary 'a' = 'A'
    0

统计相关的不要走业务主库，最好是业务和数据隔离开来，业务使用mysql，数据相关的可以是不同的方式，hbase、es等等

跨系统调用需要注意，避免长事务，事务粒度尽可能的小

主从一致：
   只读主库，从库只做failover故障转移

## 参考

[1]<http://www.dofactory.com/sql/alias>

[2]<http://www.cnblogs.com/JohnABC/p/3309144.html>

[mysql查询技巧]<http://mp.weixin.qq.com/s?timestamp=1462151732&src=3&ver=1&signature=OSVBirQ*w6WZWLcZjH6vpvdgkDEJmVj*2H8YvyHQ*kU3SMtwHC8UZjxCuA7yZG-okuygUsbvjIWjyYv0kzUA6EY59CrHnmPfjAS6z6wa9mVuYVuLetcoDSLcqObopAFRKJTeX*wR6B4E9m0GRcdt-TAvRLRKqxJMgkjiEN6WVls=>

[Character Sets and Collations Supported by MySQL]<http://dev.mysql.com/doc/refman/5.7/en/charset-charsets.html>
