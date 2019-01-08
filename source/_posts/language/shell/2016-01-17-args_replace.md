---
layout: post
title: shell参数替换
categories: language
tags: shell parameter replace
---

#### 转义字符
同c语言

#### 命令替换
\`\`  将命令的执行结果保存在变量中，如： now=\`date\`

#### 变量替换
变量替换可以根据变量的状态（是否为空、是否定义等）来改变它的值

##### ${}带冒号

|形式	|说明|
|-|-|
|${var}|变量本来的值|
|${var:-word}|如果变量 var 为空或已被删除(unset)，那么返回 word，但不改变 var 的值。
|${var:=word}|如果变量 var 为空或已被删除(unset)，那么返回 word，并将 var 的值设置为 word。
|${var:?message}|如果变量 var 为空或已被删除(unset)，那么将消息 message 送到标准错误输出，可以用来检测变量 var 是否可以被正常赋值。此替换出现在Shell脚本中，那么脚本将停止运行。
|${var:+word}|如果变量 var 被定义，那么返回 word，但不改变 var 的值。
|${var:offset}|子串,var从offset之后的子串|
|${var:offset:length}|子串，var从offset之后长度为length的子串|

#### ${}带叹号的

|形式|说明|
|-|-|
|${!prefix*}|将带有前缀为prefix的参数打印出来|
|${!prefix@}| |
|${!name[@]}|针对数组，打印出name数组有哪些下标|
|${!name[*]}||

##### ${}正则

|形式|说明|
|-|-|
|${var#pattern}|从头开始扫描，将匹配pattern正则表达式的字符过滤掉，最短匹配|
|${var##pattern}|同上，只不过是最长匹配|
|${var%pattern}|从尾开始,将匹配pattern正则表达式的字符过滤掉，最短匹配|
|${var%%pattern}|同上，只不过是最长匹配|
|${var/pattern/string}|替换，仅一次|
|${var//pattern/string}|替换，全部|

