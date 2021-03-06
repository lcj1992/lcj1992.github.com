---
layout: post
title: top
date: 2015-12-27
categories: unix
tags:
    - top
    - htop
    - glances
---

load + cpu使用率 + 内存概览 + 进程资源消耗

#### 示例

默认的展示的信息如下

    top - 08:31:11 up 7 days, 18:20,  1 user,  load average: 1.19, 0.86, 0.68
    Tasks: 188 total,   1 running, 187 sleeping,   0 stopped,   0 zombie
    Cpu(s):  6.3%us,  2.5%sy,  0.0%ni, 90.7%id,  0.0%wa,  0.0%hi,  0.4%si,  0.1%st
    Mem:   8058920k total,  7766428k used,   292492k free,     5604k buffers
    Swap:  2096440k total,  1291708k used,   804732k free,   758104k cached

      PID USER      PR  NI  VIRT  RES  SHR S %CPU %MEM    TIME+  COMMAND
    26797 *******   20   0 14.8g 6.1g 7340 S 75.8 79.7 458:59.43 java

第一行： 当前时间， 启动时间  load  参见[load_average](/2015/12/27/load_average)

第二行： 总进程数(为什么和pstree显示的数量对不上) 当前运行中的进程数  sleeping的进程数  ....

第三行： cpu相关

第四行： 内存相关


#### cpu利用率

`用户时间` User Time 即us所对应的列，表示cpu执行用户进程所占用的时间，通常情况下希望us时间的占比越高越好。

`系统时间(不要太多)` System Time 即sy所对应的列，表示cpu在内核态所花费的时间，sy的占比较高，通常意味着系统在某些方面设计的   ***不合理***，比如频繁的系统调用导致的用户态与内核态的频繁切换

`Nice时间` Nice Time即ni所对应的列，表示系统在调整进程优先级的时候所花费的时间。

`空闲时间`  Idle time 即id所对应的列，表示系统处于空闲期，等待进程运行，这个过程所占用的时间。当然，我们希望id的占比越低越好

`等待时间(不要太多)` Waiting time即wa所对应的列，表示CPU在等待I/O操作所花费的时间，系统不应该花费大量的时间来等待，否则便表示可能有某个地方设计    ***不合理***

`硬件中断时间`  Hard Irq Time 即hi所对应的列，表示系统处理硬件中断所占用的时间

`软件中断时间` Soft Irq Time 即si所对应的列，表示系统处理软件中断所占用的时间

`丢失时间`  Steal Time即st所对应的的列，是在硬件虚拟化开始流行后操作系统新增的一列，表示被强制等待虚拟CPU的时间，此时hypervisor正在另一个虚拟处理器服务，如果st占比较高，则表示当前虚拟机与该宿主山的其他虚拟机间的CPU争用较为频繁。

*   `按 1` ,查看每个核的利用率

*   `shift H` 按照线程来查看CPU的消耗情况

*   `-p`  可以指定查看的进程

#### 内存

参见 [free]

#### 进程相关

VIRT 虚拟内存

`RES` 实际占用的物理内存大小 RES = CODE + DATA

CODE Code Size (kb)

The amount of physical memory devoted to executable code, also known as the ’text resident set’ size or TRS

DATA  Data+Stack Size (kb)

The amount of physical memory devoted to other than executable code, also known as the ’data resident set’ size or DRS

#### top的加强版`htop`,`glances`

ubuntu安装

    sudo apt-get install htop
    sudo apt-get install glances

mac安装
mac上的包管理工具brew port，自行安装

	brew install htop
	brew install htop
	sudo port install glances
