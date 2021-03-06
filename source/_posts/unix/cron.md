---
layout: post
title: 定时任务
date: 2016-03-03
categories: unix
tags:
    - quartz
    - cron
---

### unix下使用crontab

1.  crontab -u 指定某一个用户
2.  crontab -e 编辑
3.  crontab -l 列举所有的定时任务
4.  crontab -r 删除定时任务
5.  ps -ef \| grep cron 查看定时任务进程是否存在
6.  /sbin/service crond start 启动定时任务进程,不同版本linux系统,命令不同
7.  定时任务在这个目录`/var/spool/cron/xx`下增加一个文件,按照cron的格式写就行
8.  cron的执行日志在`/var/log/cron`

### cron

cron可以有6到7个域

#### 概述

|域名|是否必须|允许的值|允许的特殊字符|
|--|--|--|--|
|秒|	是|	0-59|	, - * /|
|分|	是|0-59|	, - * /|
|时|	是|0-23|	, - * /|
|天|	是|1-31|	, - * ? / L W|
|月|	是|1-12 or JAN-DEC|	, - * /|
|星期|	是|1-7 or SUN-SAT|	, - * ? / L #|
|年	|`否	`|empty, 1970-2099|	, - * /|

eg:

1.  `* * * * ? *`
2.  `0/5 14,18,3-39,52 * ? JAN,MAR,SEP MON-FRI 2002-2010`

ps： jan 和JAN一样，大小写不敏感。

#### 特殊字符说明

|特殊字符|	说明|	举例|
|-----|----|---|
|*	|所有的值|
|？	|感觉和*一样啊|	Pay attention to the effects of '?' and '*' in the day-of-week and day-of-month fields!*注意什么？|
|-	|区间|	10-12 即10，11，12|
|，|	或|	10，12，14|
|/	|递增|	0/15 即 从0开始递增 0，15，30|
|L	|最后一个|	对于一月就是31号，对于二月就是29号，对于一周就是周日了。|L-2倒数第二个|
|W|最近的工作日|	15W,离15号最近的工作日，1W是周六的话，会在三号触发，不能15-17W　1.不跨月　2.作用于单个值，不作用于范围|
|LW|	这个月最后一个工作日|	发工资的日子
|#|本月的第几个星期几|	6#3，本月的第三个周五（一周从周日开始，别问为什么就是这样）|

### 参考

[1]<http://quartz-scheduler.org/documentation/quartz-2.2.x/tutorials/crontrigger>

[2]<http://linuxtools-rst.readthedocs.org/zh_CN/latest/tool/crontab.html>
