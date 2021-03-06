---
layout: post
title: 系统调用
date: 2015-12-31
categories: os
tags:
    - system_call
    - os
---

unix系统调用大致分为三类：第一类是与进程管理有关的系统调用；第二类是与文件和外设管理有关的系统调用；第三类是与系统状态有关的系统调用

#### 有关进程管理的系统调用

`fork`----建立一个进程

`exec`----执行一个文件

`wait`----等待子进程

`exit`----进程中止

`brk`----改变用户数据区大小

`sleep`----等待一段时间

`signal`----设置软中断处理程序

`kill`----发送软中断

`alarm`----在指定时间后发送软中断

`pause`----等待软中断

`nice`----改变进程优先数计算结果

`ptrace`----跟踪子进程

#### 与文件和外设管理有关的系统调用

`open`----打开文件

`close`----关闭文件

`read`----读文件

`write`----写文件

`lseek`----修改读写指针

`mknod`----建立目录或特别文件

`creat`----建立并打开文件

`link`----连接文件

`unlink`----删除文件

`chdir`----改变当前目录

`chmod`----改变文件属性

`chown`----改变文件主和用户组

`dup`----再产生一个文件描述字

`pipe`----建立并打开管道文件

`mount`----安装文件系统

`unmount`----拆卸文件系统（卷）

#### 与系统状态有关的系统调用

`getuid`----取用户号

`setuid`----设置用户号

`getgid`----取用户组号

`setgid`----设置用户组号

`time`----取日历时间

`gtty`----读取当前终端tty部分信息

`stty`----设置当前终端tty部分信息

`stat`----服务文件状态（i节点）

`sync`----使主存影像和磁盘文件信息一致
