---
layout: post
title: python调试
date: 2016-03-03
categories: language
tags: pdb
---


pdb 是 python 自带的一个包,python调试利器，
设置断点、单步调试、进入函数调试、查看当前代码、查看栈片段、动态改变变量的值等

1.  import pdb

2.  在需要设置断点的地方加入pdb.set_trace() 或者 `python -m pdb xx.py`

### 常用命令

|命令|作用|
|-|-|
|l|查看当前行的代码段|
|b|显示所有断点|
|b fileName:lineNum,condition|在某文件某行设置断点,当某个条件成立的前提下 fileName 和condition 都可不要, eg : `b xx.py:6, aa == 3`|
|n|下一步,不进入函数|
|s|下一步,可进入函数|
|c|继续执行程序|
|r|从当前函数返回,跳出当前函数|
|q|终止并退出|
|p var|打印某变量的值|
|pp expression|执行表达式的值|
|help command|查看pdb的命令|

 
### 参考

[1]<http://www.cnblogs.com/chinasun021/archive/2013/03/19/2969107.html/>

[2]<https://www.ibm.com/developerworks/cn/linux/l-cn-pythondebugger/>
 

