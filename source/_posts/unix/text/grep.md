---
layout: post
title: grep
date: 2015-12-07
categories: unix
---

## 实战

### 与、或、非
*   or

        grep -e a -e b filename
        egrep 'a|b' filename
        grep -E 'a|b' filename

*   and

        grep a filename | grep b
        grep 'a.*b' filename

*   not

        grep -v a filename

### 其他

*   -C2　    匹配的上下两行
*   -A2      匹配的之后(after)两行
*   -B2      匹配的之前(before)两行
*   --color=always  高亮显示
*   -n　--line-number   行号　    　
*   -H　--with-filename　文件名
*   -c　--count   只显示数量
*   -o       只输出匹配的字符
*   .　　　　　除了换行符以外的任意字符
*   *　　　　　0或多个
*   \\{2,5\\} 表示范围，因为shell中\{\}又特殊含义，所以需要转义

### 单引号,双引号,不加引号

1.  单引号:所见即所得
2.  双引号:如果内容中有命令,变量等,会先把变量,命令解析出结果
3.  不会将含有空格的字符串视为一个整体输出,如果内容中有命令、变量等，会先把变量、命令解析出结果，然后在输出最终内容来，如果字符串中带有空格等特殊字符，则不能完整的输出，需要改加双引号，一般连续的字符串，数字，路径等可以用
4.  $美元符, \反斜杠,`反引号,"　双引号  这四个字符在双引号中是具有特殊含义的，需要转义，其他都没有

eg: `grep "\\\\" file`   和 `grep '\\' file` 其实是一样的

## wiki　　

|操作类型|例子|描述|
|-------|----|----|
|精确匹配的字符|a A y 6 % @|字母，数字和许多特殊的字符都可以精确匹配|
| |\$ \^ \+ \\ \?|转义一些特殊的字符|
| |\n \t \r|换行、制表、返回|
|锚和宏|^|以什么开始|
| |$|以什么结尾|
|group其中任一字符|[aAeEiou]|任一字符|
|　|[^aAeiou]|任一非..字符|
| |[a-fA-F0-9]|任一的16进制字符|
| |.|除了换行符以外的任意字符|
|按之前的元素进行计数|+|一个或者更多(仅仅试用于egrep)|
| |\*|0或者更多|
| |?|0或者１|
|或|\||或者|
|分组|()|分组计数，并保存在变量中(仅仅试用于egrep)|

下边的对于grep,egrep,fgrep都适用

|选项|描述|
|---|---|
|-i|忽略大小写|
|-o|仅输出match的部分|
|-v|仅输出不满足的行|
|-c|仅输出满足的行数|
|-L|仅输出不匹配的文件名|
|-l|仅输出匹配的文件名|
|-n|输出的时候同时输出行数|
|-H|显示文件名和匹配内容|
|-r|递归|

查找空行：

        grep ^$  xx.text

### ag
使用`ag`在源代码或数据文件里检索(比grep -r更好) [github地址](https://github.com/ggreer/the_silver_searcher)

    apt-get install silversearcher-ag

    brew install the_silver_searcher

参考

[1]<http://www.wellho.net/regex/grep.html>
　　　　
[2]<http://www.shellcn.net/awk/grep_egrep_sed_awk_perl_vim_js.html>

[3]<http://blog.csdn.net/justdb/article/details/7539567>

[4]<https://github.com/ggreer/the_silver_searcher>

[shell经典笔记](http://wangxinchun.iteye.com/blog/1872951)
