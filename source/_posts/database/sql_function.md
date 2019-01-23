---
layout: post
title: 常用函数(mysql)
date: 2015-11-26
categories: db
tags:
    - sql
---




## 日期

1.  now() 获取当前时间　　　
　
2.  curdate() 获取当前日期　　
　　
3.  curtime() 获取当前时间　　　　

4.  utc_date(),utc_time(),utc_timestamp() 获取当前utc日期时间　　　

5.  date(@dt),time(@dt),year(@dt),quarter(@dt),month(@dt),week(@dt),day(@dt),hour(@dt),minute(@dt),second(@dt),microsecond(@dt)　　　　

    dt的日期，时间，年份，刻度，月份，星期，天，时，分，秒　　　　
6.  dayofweek(@dt),dayofmonth(@dt),dayofyear(@dt)　　　
　
7.  dayname(@dt),monthname(@dt)　英文名　　　　

8.  to_days(@dt)　时间转化成天　　　　

9.  to_seconds(@dt) 　时间转化成秒　　　　

10. date_add(@dt,interval 1 day) date_add(@dt,interval 1 hour) / minute,second,microsecond,week,month,quarter,year

    ps:不要使用date(create_time) + 1, 这样就可能出现20151032这样的结果,可是十月有32号么?　

11. date_add(@dt,interval '01:15:30' hour_second)　　

12. date_sub() 同date_add()　　

13. period_add(P,N),period_diff(P1,P2)　函数参数P的格式为YYYYMM或者YYMM　　

14. datediff(date1,date2),timediff(time1,time2)　　

15. str_to_date(str,format) date_format(date,format) time_format(time,format) 格式转换函数　　

## 字符串

1.  left(column_xxx,2) 第二列从左起两个字符　　

2.  right(column_xxx,2) 类上　　

3.  substr(str,pos) substr(str,pos,len)　　　

        substr(column_xx,4)　从第四个字符开始　substr(column_xx,4,2) 从第四个字符开始，截取两个　
4.  substring_index(str,delim,count)

        substring_index('www.fuck.com','.',2) 截取第二个.之前的所有字符
        substring_index('www.fuck.com','.',-2) 截取倒数第二个.之前的所有字符
        substring
5.  length() 字符串长度　　　　

6.  locate(substr,str) locate(substr,str,5) 返回子串substr在字符串str第一(5)个出现的位置，如果substr不是在str里面，返回0　　

7.  instr(str,substr)　和locate一样，除了两个参数顺序相反。

8.  ltrim(column_xx),rtrim(column_xx),trim(column_xx) 去除前置空格，去除后置空格，去除空格

9.  拼接字段,concat()将多个逗号隔开的字符串链接起来

## 其他

* 从文件中导数据

        LOAD DATA LOCAL INFILE '~/Music/xx.csv'
        INTO TABLE tb_xx
        FIELDS TERMINATED BY ','
            ENCLOSED BY '"'
        LINES TERMINATED BY '\n'
        IGNORE 1 LINES
* mysql和java时间类型转换

|mysql|java|
|-|-|
|date|java.sql.Date|
|Datetime|java.sql.Timestamp|
|Timestamp|java.sql.Timestamp|
|Time|java.sql.Time|
|Year|java.sql.Date|

1.  ***mysql中Datetime范围(1000-01-01 00:00:00  9999-12-31 23:59:59)***
2.  ***Timestamp范围(1970-01-01 00:00:00  2037-12-31 23:59:59)***
3.  ***两者都对应java的Timestamp,范围(1970-01-01 00:00:00  2037-12-31 23:59:59)***

*  转换

        CASE
            WHEN p.gender = 1 THEN 'boy'
            WHEN p.gender = 2 THEN 'girl'
        END
