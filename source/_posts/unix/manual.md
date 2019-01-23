---
layout: post
title: 学会使用手册
date: 2015-12-22
categories: unix
tags:
    - man
    - unix
---

### apropos　

搜索手册页名称和描述

-r --regex　正则匹配，默认的　

-e --exact  精确匹配

查看which命令是做什么用的

    apropos -e which

### manual(手册)

| 快捷键| 作用  |
| -----  | ----- |
| NAME | 命令简写的全称，一句话表明该命令的作用 |
| SYNOPSIS | 概述，对命令的使用方法做一个总结性的描述 |
| DESCRIPTION | 对NAME做进一步的补充，描述这个命令都干了什么 |
| OPTIONS | 对命令各个option做详细的描述 |
| OUTPUT | 有些命令的输出包含的字段也很多，对输出的各个项进行解释 |
| EXAMPLES | 示例 |

除此之外还有AUTHOR,REPORTING BUGS,COPYRIGHT,SEE ALSO等,man手册一般放在/usr/share/man/man1/xx.gz

### 什么哪里

*	`whatis (1)`           - display one-line manual page descriptions　这个命令是什么
*	`whereis (1)`          - locate the binary, source, and manual page files for a command
*	`w (1)`                - Show who is logged on and what they are doing.
*	`whoami (1)`           - print effective userid
*	`whois (1)`           - client for the whois directory service
*	`which (1)`            - locate a command

ubuntu
/bin/xx　系统自带的
/usr/bin/xx　安装的不需要root权限的
/usr/sbin/xx 安装的需要root权限的

mac
/usr/local/bin/ brew安装的软件都在这

### 实用至上

手册里都有，为什么还要专门搞个地方写几篇文章来记linux命令呢？

1.线上故障了，你去现查，找死

2.你平时设计的业务量也就那么点，好多高级功能，你根本都不需要，或者说不适用

所以就有这个目录下的这些文章，旨在记录一些日常工作中使用频率较高的命令，一个字实用至上,man手册仅仅是我们的最后一根救命稻草．


### 参考

[1]<https://github.com/lcj1992/the-art-of-command-line/blob/master/README-zh.md>

[2]<http://linuxtools-rst.readthedocs.org/zh_CN/latest/>

[3]<http://ss64.com/bash/>

[4]<http://man.linuxde.net/par/3>
