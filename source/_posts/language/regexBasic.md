---
layout: post
title: 正则基础
date: 2015-12-08
categories:
    - regex
---


抄自：http://deerchao.net/tutorials/regex/regex.htm

## 元字符

|代码|说明|
|---|---|
|.|匹配除换行符以外的任意字符|
|\w|匹配字母或者数字或下划线或汉字|
|\W|匹配任意不是字母，数字，下划线，汉字的字符|
|\s|匹配任意的空白字符|
|\S|匹配任意非空白字符|
|\d|匹配数字|
|\D|匹配任意非数字的字符|
|\b|匹配单词的开始或结束|
|\B|匹配不是单词开头或结束的位置|
|^|匹配字符串的开始|
|$|匹配字符串的结束|
|[^x]|匹配非x的任意字符|
|[^aeiou]|匹配除aeiou的任意字符|

如果要查找的时元字符本身，就需要进行转义 \\. \\*

1.  \ba\w*\b 　　以字母a开头的
2.  \d+ 　　　　　匹配１个或更多连续的数字
3.  ^\d{5,12}$　　5-12位的数字

## 重复
*   重复0,1,n次

|代码/语法|说明|
|--|--|
|\*|重复0次或更多次|
|+|重复１次或更多次|
|?|重复0次或1次|
|{n}|重复n次|
|{n,}|重复n次或更多次|
|{n,m}|重复n到m次|

1.  Windows\d+　匹配Windows后面跟1个或更多数字
2.  ^\w+　匹配一行的第一个单词或者整个字符串的第一个单词，具体匹配那个看选项设置　　

<h2 id="branch">分支条件</h2>
1.  分支条件指的是有几种规则，如果满足了其中任意一种规则都应该当做匹配，具体方法是用\|把不同的规则分隔开。
2.  从左到右测试每个分支条件，如果满足了某个分支的话，就不会去再管其他的条件了。

## 分组
*   如果想要重复多个字符，可以用小括号来制定子表达式(分组),然后你就可以指定这个子表达式的重复次数了，你也可以对子表达式进行其他依稀操作　　　　　
*   (\d{1,3}\\.}{3}\d{1,3})　\d{1,3}匹配１到3位的数字,(\d{1,3}\\.){3})匹配三位数字加上一个英文句号(这个整体也就是这个分组)重复三次，最后...

<h2 id="backwardReference">后向引用</h2>
*  使用小括号指定一个子表达式后，匹配这个子表达式的文本(也就是此分组捕获的内容)可以在表达式或其他程序中作进一步的处理。
    默认情况下，每个分组会自动拥有一个组号，规则是：从左到右，以分组的左括号为标志，第一个出现的分组的组号为１，第二个为2，以此类推。

|分类|代码/语法|说明|
|---|---|---|
|捕获|(exp)|匹配exp，并捕获文本到自动命名的组里|
| |(?<name>exp)|匹配exp，并捕获文本到名称为name的组里，也可以写成(?'name'exp)|
| |(?:exp)|匹配exp，不捕获匹配的文本，也不给此分组分配组号|
|零宽断言|(?=exp)|匹配后面跟的是exp的位置|
| |(?<=exp)|匹配前面是exp的位置|
| |(?!exp)|匹配后面跟的不是exp的位置|
| |(?\<!exp)|匹配前面不是exp的位置|
|注释|(?#comment)|这种类型的分组不对正则表达式的处理产生任何影响，用于提供注释让人阅读|

*   \b\w+(?=ing\b)　I'm singing while you're dancing　会匹配sing和danc
*   (?\<=\bre) reading a book 会匹配ading
*   \d{3}(?!\d)　匹配三位数字，而且这三位数字的后面不能是数字
*   (?\<![a-z])\d{7} 匹配前面不是小写字母的７位数字

## 贪婪与懒惰
*   不加?是贪婪，加?是懒惰

|代码/语法|说明|
|--|--|
|\*?|重复任意次,但尽可能少重复|
|+?|重复1次或更多次，但尽可能少重复|
|??|重复0次或1次，但尽可能少重复|
|{n,m}?|重复n到m次，但尽可能少重复|
|{n,}?|重复n次以上，但尽可能少重复|

*   a.*?b　如果应用于aabab的话，它会匹配aab和ab
