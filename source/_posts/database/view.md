---
layout: post
title: 视图
date: 2015-12-27
categories: db
tags: view
---


#### 定义

存起来的查询命令，只是个查询，没有数据，动态查询

在数据库理论中，一个视图是数据库上的一个存储的查询的结果集。

预先建立的查询命令是保存在数据库的字典中，不像关系数据库中一般的基表，一个视图不是真实的schema的一部分，它是动态的计算数据库中的数据得出的一个结果集。基表变了它也会跟着变。

视图和它的源查询是等价的。

eg:

创建视图:

    CREATE VIEW accounts_view AS
        SELECT 
            name,
            money_received,
            money_sent,
            (money_received - money_sent) AS balance,
            address
        FROM
            table_customers c
                JOIN
            accounts_table a ON a.customer_id = c.customer_id

使用视图:

    Simple query
    ------------
    SELECT name,
           balance
      FROM accounts_view

执行过程:

    Preprocessed query:
    ------------------
    SELECT name,
           balance
      FROM (SELECT name,
                   money_received,
                   money_sent,
                   (money_received - money_sent) AS balance,
                   address,
                    ...
              FROM table_customers c JOIN accounts_table a
                   ON a.customer_id = c.customer_id        )
    
#### 参考

[wikipedia]<https://en.wikipedia.org/wiki/View_(SQL)>
