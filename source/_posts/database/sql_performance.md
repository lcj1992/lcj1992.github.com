---
layout: post
title: 数据库查询性能优化
date: 2015-12-27
categories: db
tags:
    - performance
    - sql
---

#### 慢日志

查看是否启用慢日志

    show variables like 'log_slow_queries';

查看慢于多少秒的sql会记录到慢日志中

    show variables like ‘long_query_time’;

通过mysql的配置文件my.cnf，可以修改慢日志的相关配置

#### 合理使用索引

1. 业务上具有唯一特性的字段，即使是多个字段的组合，也必须建成唯一索引。
2. 超过三个表禁止 join。需要 join 的字段，数据类型必须绝对一致;多表关联查询 时，保证被关联的字段需要有索引。
3. 能用覆盖索引就用覆盖索引，避免回表
4. 利用延迟关联或者子查询优化超多分页场景，mysql查询limit offset,n.是先查出offset+n，然后舍弃掉前offest行，返回n行。先查询主键，基于主键进行数据查询，可以是子查询也可以是join表
```
select  * from mtp_book_info where book_id >= (select book_id from mtp_book_Info order by book_id  desc limit 1000000, 1) order by book_id desc  limit 10;

select * from mtp_book_info as book1 join (select book_id from mtp_book_info order by book_id desc limit 1000000, 10) as book2 where  book1.book_id = book2.book_id ;
```
5. 如果有order by的场景，请注意利用索引的有序性。order by最后的字段是组合 索引的一部分，并且放在索引组合顺序的最后，避免出现 file_sort 的情况，影响查询性能。 正例:where a=? and b=? order by c; 索引:a_b_c ???
6. SQL 性能优化的目标:至少要达到range(索引范围检索)级别，要求是ref（普通的索引）级别，如果可以是consts(主键或唯一索引)最好。
7. 建组合索引的时候，区分度最高的在最左边。组合索引和多个单列索引，当出现索引合并时，考虑建组合索引
8. 防止因字段类型不同造成的隐式转换，导致索引失效。
9. in 操作能避免则避免，若实在避免不了，需要仔细评估 in 后边的集合元素数量，控 制在 1000 个之内。
10. 使用 ISNULL()来判断是否为 NULL 值。注意:NULL 与任何值的直接比较都为 NULL。
11. 查询条件中含有函数或表达式用不到索引
12. Index Selectivity = Cardinality / #T ， 索引的选择性（Selectivity），是指不重复的索引值（也叫基数，Cardinality）与表记录数（#T）的比值，索引的选择性较低不建议建立索引
13. 前缀索引


#### 最左前缀原则

*   必须按照顺序（即查询条件中必须出现按照索引声明顺序的列才生效），如idx_site_status_channel

查询条件中 只有site 或者  site，status （status，site）或者site，channel，status 都会走这个索引，从左到右。

例子：

select * from task_info where status = 3 and stie='CA'; 会用到索引

select * from task_info where channel = 3 and status = 'CA'; 不会用到

*   如果查询中某个列使用了范围查询，则其右边所有的列都无法使用索引进行查询优化


### 计算qps,tps

如果数据库中存在比较多的myisam表，则计算还是questions 比较合适。
如果数据库中存在比较多的innodb表，则计算以com_*数据来源比较合适。

#### 方法一

基于 questions  计算qps,基于  com_commit  com_rollback 计算tps

`questions` = show global status like 'questions';

`uptime` = show global status like 'uptime';

`com_commit` = show global status like 'com_commit';

`com_rollback` = show global status like 'com_rollback';

1.`qps` =questions/uptime

2.`tps` = (com_commit + com_rollback)/uptime


#### 方法二

基于 com_* 的status 变量计算tps ,qps

使用如下命令：

`show global status where variable_name in('com_select','com_insert','com_delete','com_update');`

获取间隔1s 的 com_*的值，并作差值运算

`del_diff` = (int(mystat2['com_delete'])   - int(mystat1['com_delete']) ) / diff

`ins_diff` = (int(mystat2['com_insert'])    - int(mystat1['com_insert']) ) / diff

`sel_diff` = (int(mystat2['com_select'])    - int(mystat1['com_select']) ) / diff

`upd_diff` = (int(mystat2['com_update'])   - int(mystat1['com_update']) ) / diff

#### 参考    

[1]<http://coolshell.cn/articles/1846.html>

[2]<http://blog.itpub.net/22664653/viewspace-767265/>
