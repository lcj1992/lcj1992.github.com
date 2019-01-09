---
layout: post
title: tr
date: 2015-12-25
categories: unix
tags : tr
---


tr translate

|选项|作用|
|-|-|
|-C||
|-c|用字符串1中字符集的补集替换此字符集|
|-s|删除所有重复出现字符序列，只保留第一个|
|-d|从输入中删除String1中包含字符|

指定字符串1或字符串2的内容时，只能使用单字符或字符串范围或列表

使用tr命令“统一”字母大小写 
    
    cat file | tr [a-z] [A-Z] > new_file
    tr "[:lower:]" "[:upper:]" < file1

将文本转成单词list,,一行一个

    tr -cs "[:alpha:]" "\n" < file1
    
删除文件file中出现的换行'\n'、制表'\t'字符

    cat file | tr -d "\n\t" > new_file
    
删除“连续着的”重复字母，只保留第一个

    cat file | tr -s [a-zA-Z] > new_file
删除空行   

    cat file | tr -s "\n" > new_file

把路径变量中的冒号":"，替换成换行符"\n"
    
    echo $PATH | tr -s ":" "\n"
