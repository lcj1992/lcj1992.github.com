---
layout: post
title: find实用
date: 2014-11-13
categories: unix
tags:
    - find
    - locate
---

在文件树种查找文件，并做相应处理

        find pathname -options [-print -exec -ok ...]

命令参数

|参数|作用|
|-|-|
|pathname|路径名|
|-print|find命令将匹配的文件输出到标准输出中|
|-exec|对匹配的文件执行命令,相应的命令形式为 `cmd {} \;`，注意{}和\;之间有个空格|
|-ok|同-exec，不过每个文件需要确认|

eg:
        find . -size +100000000c -exec ls -l {} \;

命令选项
|选项|作用|
|-|-|
|-name|按照文件名|
|-perm|按照文件权限来查找文件|
|-user|按照文件属主来查找文件|
|-group|按照文件属组来查找文件|
|-mtime -n + n|按照文件的更改时间来查找文件 -n n天以内，+n n天之前，类似还有-atime  -ctime|
|-newer file !file|查找更改时间比file1新，但比file 2旧的文件|
|-type|按文件类型来查找[类型参考](/2014/11/13/file)|
|-size n|查找文件长度为n块的文件，带有c时表示文件长度以字节计|
|-follow|如果遇到符号链接，就跟踪至链接所指向的文件|
