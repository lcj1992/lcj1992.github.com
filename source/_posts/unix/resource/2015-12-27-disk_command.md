---
layout: post
title: 磁盘命令
categories: unix
tags: du df
---

    lcj@lcj:~$ df -h
    文件系统        容量  已用  可用 已用% 挂载点
    /dev/sda1       227G   76G  141G   35% /
    none            4.0K     0  4.0K    0% /sys/fs/cgroup
    udev            3.9G  4.0K  3.9G    1% /dev
    tmpfs           790M  1.3M  788M    1% /run
    none            5.0M     0  5.0M    0% /run/lock
    none            3.9G   32M  3.9G    1% /run/shm
    none            100M   80K  100M    1% /run/user

磁盘的剩余空间.-h 按单位格式化输出

如果需要查看具体目录所占用的空间，分析大文件所处位置，可使用du命令来进行查看

    lcj@lcj:~/bin$ du -m | sort -nr  | head -10
    10644	.
    8627	./openjdk
    5757	./openjdk/build
    3036	./openjdk/build/linux-amd64
    2447	./openjdk/build-debug
    1369	./openjdk/build/linux-amd64/hotspot
    1238	./openjdk/build/linux-amd64/hotspot/outputdir
    1237	./openjdk/build/linux-amd64/hotspot/outputdir/linux_amd64_compiler2
    1181	./openjdk/build/linux-amd64/hotspot/outputdir/linux_amd64_compiler2/product
    1044	./openjdk/build/hotspot


-h 按文件大小单位的格式化输出 　　

-m 兆

    lcj@lcj:~$ iostat -d -k
    Linux 3.13.0-32-generic (lcj) 	2015年12月27日 	_x86_64_	(4 CPU)

    Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
    sda              16.23       176.98       239.19   49857235   67385332


-d 表示查看磁盘使用情况

-k 表示以KB为单位显示

Device 设备名称
tps 每秒处理的I/O请求数
KB_read/s表示每秒从设备读取的数据量
KB_wrtn/s表示每秒向设备写入的数据量
KB_read 表示读取的数据总量
KB_wrtn 表示写入的数据总量



