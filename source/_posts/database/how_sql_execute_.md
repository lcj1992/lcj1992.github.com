---
layout: post
title: 一条sql的执行（mysql）
date: 2016-07-16
categories: db
tags: explain select mysql
---




## select sql执行顺序 

    (8)SELECT (9)DISTINCT
    <select_list>
    (1)FROM <left_table>
    (3)　<join_type> JOIN <right_table>
    (2)　 ON <join_condition>  
    (4)WHERE <where_condition>
    (5)GROUP BY <group_by_list>
    (6)WITH {CUBE | ROLLUP}
    (7)HAVING <having_condition>
    (10)ORDER BY <order_by_list>
    (11)<LIMIT_specification>

执行顺序:

`笛卡尔积---on---join类型---where---group by---with{cube | rollup}---having---select---distinct---order by---limit`

ps:

limit works on MySQL and PostgreSQL, top works on SQL Server, rownum works on Oracle.

### 细解 

1.  on vs where
    两者效果可能一样(内连接)，但on是连接两表做笛卡尔积时的连接条件，where连接之后的筛选条件。

2.  where vs having

    1.  where 在对查询结果进行分组前(`group by 之前`)，将不符合where条件的行去掉，即在分组之前过滤数据， ***where条件中不能包含聚集函数***，使用where条件过滤出特定的行。

    2.  having 筛选满足条件的组(`group by 之后`,having是专门搭配group by干活的)，即在分组之后过滤数据，条件中经常包含聚集函数，使用having条件过滤出特定的组，也可以使用多个分组标准进行分组。

3.  join、left join、right join　　

    eg table_a,table_b

    1.  left join 以左为准，右可以为空, 记录数>=table_a总数
    2.  right join 以右为准，左可以为空，记录数>=table_b总数

4.  group by

    1.  先对某一列或多列进行分组，然后在分组内进行相应的操作，一般和count，max等聚集函数一起使用。

    2.  聚集函数与列不能同时出现在select子句中，除非这个列是group by子句的分组列,参见下错误例子

    3.  当查询语句中有group by子句时，聚集函数作用的对象是一个分组，而不是整个查询的结果,没有group by的话是整个查询结果

        select DepartmentId,Name,count(Salary) from Employee group by Name;

        Error Code: 1055. Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'test.Employee.DepartmentId' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by

5.  order by

    对某一列进行排序,发生在select，distinct之后

6.  limit

    取出结果的几个 eg：limit 1,2从1开始的2个。  查询结果是从0 开始的,db会扫出这n＋m条记录，然后筛选。所以分页慎用limit m,n

7.  union和union all

    跨库查询，union会去重，union all 不会去重

8.  group by 之后，可以通过加order by null避免文件排序

## where条件在数据库中提取

SQL语句中的where条件，使用以上的提取规则，最终都会被提取到`Index Key` (`First Key` & `Last Key`)，`Index Filter`与`Table Filter`之中。

1.  Index Key
    1.  Index First Key，只是用来定位索引的起始范围，因此只在索引第一次Search Path(沿着索引B+树的根节点一直遍历，到索引正确的叶节点位置)时使用，一次判断即可，
        1.  从索引的第一个键值开始，检查其在where条件中是否存在，若存在并且条件是=，则将对应的条件加入Index First Key之中，继续读取索引的下一个键值，使用同样的提取规则
        2.  若存在并且条件是>=或者>，则将对应的条件加入Index First Key中，同时终止Index First Key的提取；若不存在，同样终止Index First Key的提取。
    2.  Index Last Key，用来定位索引的终止范围，因此对于起始范围之后读到的每一条索引记录，均需要判断是否已经超过了Index Last Key的范围，若超过，则当前查询结束；

        1.  从索引的第一个键值开始，检查其在where条件中是否存在，若存在并且条件是=，则将对应条件加入到Index Last Key中，继续提取索引的下一个键值，使用同样的提取规则
        2. 若存在并且条件是 <或<= ，则将条件加入到Index Last Key中，同时终止提取；若不存在，同样终止Index Last Key的提取。

2.  Index Filter
    用于过滤索引查询范围中不满足查询条件的记录，因此对于索引范围中的每一条记录，均需要与Index Filter进行对比，若不满足Index Filter则直接丢弃，继续读取索引下一条记录；

        1.  从索引列的第一列开始，检查其在where条件中是否存在：若存在并且where条件仅为 =，则跳过第一列继续检查索引下一列，下一索引列采取与索引第一列同样的提取规则
        2.  若where条件为 >=、>、<、<= 其中的几种，则跳过索引第一列，将其余where条件中索引相关列全部加入到Index Filter之中
        3.  若索引第一列的where条件包含 =、>=、>、<、<= 之外的条件，则将此条件以及其余where条件中索引相关列全部加入到Index Filter之中
        4.  若第一列不包含查询条件，则将所有索引相关条件均加入到Index Filter之中。

3.  Table Filter
    最后一道where条件的防线，用于过滤通过前面索引的层层考验的记录，此时的记录已经满足了Index First Key与Index Last Key构成的范围，并且满足Index Filter的条件，回表读取了完整的记录，判断完整记录是否满足Table Filter中的查询条件，同样的，若不满足，跳过当前记录，继续读取索引的下一条记录，若满足，则返回记录，此记录满足了where的所有条件，可以返回给前端用户。

### 源码分析

入口函数：
1. sql_select.cc#handle_query
    1. 单一查询
        1. 设置limit
        2. prepare
    2. join或者子查询
2. optimize
    1. 单一查询
        select-> optimize
    2. join或子查询
        unit-> optimize
3. 是否是explain语句
    1. 是 执行explain_query
    2. 否 是否是单一查询
        1. 是 select->join->exec()
        2. 否 unit->exec()
connection_handler_per_thread.cc#do_command

sql_parse.cc#mysql_execute_command
execute_sqlcom_select



收集一下问题：
1. 查询优化器如何优化
    1. 联合索引和单列索引如何选择
    2. 两个单列索引是选择索引合并还是选一个选择性较大的，然后using where
    3. ICP

## 参考

[SQL中的where条件，在数据库中提取与应用浅析](http://hedengcheng.com/?p=577)
