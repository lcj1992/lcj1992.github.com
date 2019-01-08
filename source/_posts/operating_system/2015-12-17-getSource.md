---
layout: post
title: ubuntu获取源码
categories: os linux
---

### 获取工具源码
以ls为例,会被下载到当前目录

       lcj@lcj:~$ which ls
       /bin/ls
       lcj@lcj:~$ dpkg -S /bin/ls
       coreutils: /bin/ls
       lcj@lcj:~$ sudo apt-get source coreutils








