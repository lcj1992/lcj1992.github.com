---
layout: post
title: 索引的数据结构（mysql）
date: 2017-10-10
categories: db
tags: index
---




## 索引的数据结构

### myisam

![myisam_pri](/images/database/myisam_pri.png)

1.  使用[b＋树](https://en.wikipedia.org/wiki/B%2B_tree)
2.  **索引文件和数据文件是分离的。** 索引文件仅保存数据记录的地址，索引的叶子节点的data域存放的是数据记录的地址
(上图`标红1处`key为15的叶子节点，只保存数据域的指针0x07,标红2处,是存放数据记录的位置)
3.  **主索引和二级索引结构上没有任何差别，只是主索引要求key必须是唯一的**

### innodb

![innodb_pri](/images/database/innodb_pri.png)
![innodb_second](/images/database/innodb_second.png)

1.  同样是b+树
2.  数据文件同样也是索引文件，数据文件本身就是按B+Tree组织的一个索引结构，叶子节点data域保存完整的数据记录(primary key图中`标红1处`,key为15的叶子节点，数据域就是数据记录)，这个索引的key是数据表的主键，因此innodb的表数据文件本身就是主索引。
3.  innodb主索引，叶节点包含了完整的数据记录，这种索引叫做`聚集索引`
4.  innodb的辅助索引data域存储相应记录主键的值而不是数据记录的地址（`标红2处`,以col3建了个索引，key为col3，value是其对应的主键）。
5.  联合索引和最左前缀匹配
6.  主键不一定是一个字段哟！

ps: 因为innodb的数据文件本身要按主键聚集，所以innodb要求表必须有主键（myisam可以没有），如果没有显式指定，则mysql系统会自动选择一个可以唯一标识数据记录的列作为主键，如果不存在这种列，则mysql自动为innodb表生成一个隐含字段作为主键，这个字段长度为6个字节，类型为长整形。
