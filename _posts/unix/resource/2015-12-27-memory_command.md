---
layout: post
title: 内存命令
categories: unix
tags: free memory　vmstat
---

#### free

    lcj@lcj:~/bin$ free -m
                 total       used       free     shared    buffers     cached
    Mem:          7891       5280       2610        682        356       1437
    -/+ buffers/cache:       3486       4404
    Swap:         8095       1064       7031

-m 表示以MB为单位

Linux的内存包括物理内存Mem和虚拟内存swap

|字段|说明|
|-|-|
|total |内存总共的大小
|used|已使用内存的大小
|free|可使用的内存大小
|shared|多个进程共享的内存空间大小
|buffers| 缓冲区的大小
|cached |缓存的大小

linux的内存管理机制与windows有所不同，其中有一个思想便是`内存利用率最大化`，内核会将剩余的内存申请为cached，而cached不属于free范畴。
因此，当系统运行时间较长时，会发现cached这块区域比较大，对于有频繁文件读/写操作的系统，这种现象更加明显。但是

`free的内存小，并不代表可用的内存小。`

当程序需要申请更更大的内存时，如果free内存不够，系统会将部分cached或buffers内存回收，回收的内存再分配给应用程序。
因此，linux可用于分配的内存不仅仅只有free的内存。可以看

`free命令显示的第三行，也就是-/+buffers/cache 对应的行，这一行将内存进行了重新计算，used减去buffers和cached占用的内存，而free则加上了buffers和cached对应的内存`。

对于应用来说，更值得关注的应该是虚拟内存`swap的消耗`，swap内存使用过多，表示物理内存已经不够用了，操作系统将本应该物理内存存储的一部分内存页调到磁盘上，以腾出足够的空间给当前的进程使用。
当其他进程需要运行时，再从磁盘将内存的页调度到物理内存当中，以恢复进程的运行。而这个调度的过程，则会产生swap I/O，如果swap I/O较为频繁，将严重影响系统的性能。

#### vmstat
vmstat可以查看swap I/O的情况

    lcj@lcj:~/bin$ vmstat
    procs -----------memory---------- ---swap-- -----io---- -system-- ------cpu-----
     r  b   swpd   free   buff  cache   si   so    bi    bo   in   cs us sy id wa st
     0  0 1090036 3014664 366924 1414400   35   31   221   299  176  148 17 22 61  0  0


其中swap列的si表示每秒从磁盘交换到内存的数据量，单位是KB/s，so表示每秒从哦过内存交换到磁盘的数据量，单位也是KB/s


#### 参考

[Linux内存中的Cache真的能被回收么?](http://liwei.life/2016/04/26/linux%E5%86%85%E5%AD%98%E4%B8%AD%E7%9A%84cache%E7%9C%9F%E7%9A%84%E8%83%BD%E8%A2%AB%E5%9B%9E%E6%94%B6%E4%B9%88%EF%BC%9F/)

[cache buffer](https://serverfault.com/questions/85470/meaning-of-the-buffers-cache-line-in-the-output-of-free)