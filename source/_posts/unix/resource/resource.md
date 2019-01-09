---
layout: post
title: 系统资源
date: 2015-11-27
categories: unix
tags: top netstat performance
---

*   w
*   top,htop(top增强版，高亮),atop（不同于前两者，按日记录记录进程信息供以后分析）
*   mytop 监控mysql　
*   powertop
*   iotop
*   vmstat (这种统计命令一般都可以指定时间　vmstat 1 10 1为间隔时间　10是重复次数)
*   dstat　多功能工具生成系统资源
*   iftop 按host统计网卡的带宽使用
*   jnettop  以相同的方式来监测网络流量但比 iftop 更形象
*   nethogs
*   iptraf
*   ngrep
*   traceroute
*   iptstate
*   vnstat
*   nmap
*   mtr
*   dstat
*   iostat　统计cpu和设备，分区与网络的输入输出数据
*   netstat　打印网络连接，路由表，网卡信息，伪装连接
*   mpstat 统计处理器相关的数据
*   pstree
*   sar　收集，统计，保存系统活动信息
*   ethtool
/2015/12/26/jvm_tools)


     Performance tools　　

            ifstat(1), iftop(8), iostat(1), mpstat(1), netstat(1), nfsstat(1), nstat, vmstat(1), xosview(1)

     Debugging tools　　

            htop(1), lslk(1), lsof(8), top(1)

     Process tracing　　　

            ltrace(1), pmap(1), ps(1), pstack(1), strace(1)

     Binary debugging　　

           　ldd(1), file(1), nm(1), objdump(1), readelf(1)

     Memory usage tools　　

            free(1), memusage, memusagestat, slabtop(1)

     Accounting tools　　

            dump-acct, dump-utmp, sa(8)

     Hardware debugging tools　　

            dmidecode, ifinfo(1), lsdev(1), lshal(1), lshw(1), lsmod(8), lspci(8), lsusb(8), smartctl(8), x86info(1)

     Application debugging　　

            mailstats(8), qshape(1)

     Xorg related tools　　

            xdpyinfo(1), xrestop(1)

     Other useful info　　

            collectl(1), proc(5), procinfo(8)

参考

[80 多个 Linux 系统管理员的监控工具]<http://www.codeceo.com/article/80-more-linux-monitor-tools.html>
