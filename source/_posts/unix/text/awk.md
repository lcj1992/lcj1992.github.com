---
layout: post
title: awk
date: 2015-12-25
categories: unix
tags:
    - awk
---

## 原理图

![原理图](/images/unix/awkFlow.jpg)

awk提供一个类似于编程的开放环境，让你能够自定义文本处理的规则，修改和重新组织文件中的内容。awk在流编辑法方面比sed更为先进的是：它提供一种编程语言而不是一组文本编辑的命令，在编程语言的内部，可以定义保存数据的变量，使用算术和字符串操作函数对数据进行运算，支持结构化编程概念，能够使用if和循环语句等。

        awk  [option] 'pattern { action }' file　　

option为命令的选项，pattern为行匹配规则，action为执行的具体操作：如果没有pattern，则对所有行执行action，而如果action，则打印所有匹配的行，file为输入的文件。eg:

        awk 'BEGIN {FS=":";print "---header---"} /mysql/  {print $1}  END {print "---footer---"}' /etc/passwd

## 内建变量

内建变量

|变量名|含义|
|---|---|
|$1|当前记录的第一个字段，字段间有FS分隔，$2~$n类似|
|$0|当前记录（这个变量存放着整个行的内容）|
|NF|当前记录中的字段个数，就是有多少列，输出最后一列|
|NR|已读出的记录数，就是行号，从1开始，如果有多个文件的话，这个值也是不断累加|
|FNR|已读出的记录数，与NR不同，这个值会是各个文件自己的行号|
|FS|输入字段分隔符，默认是空白符|
|RS|输入的记录分隔数,默认为换行符|
|OFS|输出字段分隔符，默认空格|
|ORS|输出的记录分隔符，默认换行|
|FILENAME|当前输入文件的名字|
|length($0)|当前行的长度|

## 例子

1.打印第1，4列，其中$0是所有列　　

`awk '{print $1,$4}' netstat.txt`

2.格式化输出，跟c的格式化一样（除没有逗号了）%-8s：左对齐八个字节，不足空格补齐　　


`awk '{printf "%-8s %-8s %-8s %-22s %-15s\n",$1,$2,$3,$4,$5}' netstat.txt`

3.条件打印 比较运算符  ==,!=,>,<,>=,<= 或与 &&,\|\|　　

`awk '$2==0 && $6=="TIME_WAIT"' netstat.txt`

4.输出 第三列值为0 且第六列值为ESTABLISHED或者第一行，打印总行号，行号，第四列，第五列，第六列

`awk '$3==0 && $6=="ESTABLISHED" || NR==1 {printf "%02s %s %-20s %-20s %s\n",NR,FNR,$4,$5,$6}' netstat.txt`

5.输出第三列值大于0 ，打印所有列

`awk '$3> 0 {print $0}' netstat.txt`

6.模式匹配 匹配第六列包含WAIT的行或者第一行，打印行号，第四列，第五列，第六列,并以制表符为分隔符

`awk '$6 ~ /WAIT/ || NR==1 {print NR,$4,$5,$6}' OFS="\t" netstat.txt`

7.模式匹配，匹配第六列包含FIN或则会TIME的行....

`awk '$6 ~ /FIN|TIME/ || NR==1 {print NR,$4,$5,$6} OFS="\t" netstat.txt`

8.第六列不包含WAIT的行

`awk '$6 !~ /WAIT/ || NR== 1{print NR,$4,$5,$' OFS="\t" netstat.txt`

9.不包含WAIT的行

`awk '!/WAIT/' netstat.txt`

10.指定分隔符为:

`awk 'BEGIN{FS=":"}{print $1,$3,$6}' /etc/passwd`
`awk -F :  '{print $1,$3,$6}' /etc/passwd`

11.指定分隔符为;或者:

`awk -F '[;:]' '{print $1,$3,$6}' /etc/passwd`

12.不输出第一列，格式化输出到文件中，以第六列命名

`awk 'NR!=1  {print > $6}' netstat.txt`

13.相加某列

`awk '$3==12 {sum += $4;month=$3} END {print month,sum/30}' ticket_count `
`awk 'NR < 9 {sum += $0} END {print sum}}`

14.去重

`awk '{if (!seen[$0]++) { print $0; } }' netstat.txt`

`netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr`

-f 选项来指定包含文本处理程序的脚本文件

15.打印最后一列

`awk '{print $NF}' file`
