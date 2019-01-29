---
layout: post
title: 网络
date: 2015-03-20
categories: unix
tags:
    - unix
---

### iftop

![iftop](/images/network/iftop.png)

*   TX：发送流量

*   RX：接收流量

*   TOTAL：总流量

*   Cumm：运行iftop到目前时间的总流量

*   peak：流量峰值

*   rates：分别表示过去 2s 10s 40s 的平均流量

### tcpdump

研究网络协议必备

#### wiki

|命令选项|作用|
|-|-|
|-i any|  监听所有的界面。这样你就知道是不是有流量产生|
|-n| 不要解决主机名，以IP数字形式显示主机|
|-nn|不要解析主机名或端口名|
|-X|同时以十六进制和ASCII字符显示包的数据(packet) `网络层+传输层`|
|-XX|同-X,但也会显示Ethernet头部(frame) `链路层+网络层+传输层`|
|-v, -vv, -vvv|增量输出得到的包信息 v越多信息越丰富|
|-c|抓取 x 个包后就停下|
|-s|设置显示前多少个字节的包内容。使用-s0获得所有的数据，除非你要故意获取|
|-S|打印绝对序号|
|-e|同时得到Ethernet头部|
|-q|显示少一点协议信息|
|-E|解密IPSEC流量提供加密密钥|
|-A|以ASCII打印每个包,对于网页抓取特别好用|

ps:tcpdump 4.0的snaplength的长度从68字节改成了96字节，这样你就可以看到多些内容了。但仍然看不到所有的内容，指定 -s 1514 得到包的所有内容
eg:

    tcpdump -nS
    tcpdump -nnvvS
    tcpdump -nnvvXSs 1514

#### 语法

使用表达式可以让你略去各种各样的流量而只得到你真正关注的。掌握表达式并且会创造性地使用组合技巧才使你真正发挥tcpdump的力量。有三种主要的表达式: type, dir 和 proto.

***1.主要的表达式:***

type: `host`,`net`,`port`,`portrange`

dir: `src`,`dst`

proto:`tcp`,`udp`,`icmp`...

`less` ,`greater`,'>' '>='

eg:

    tcpdump host 1.2.3.4
    tcpdump src 2.3.4.5
    tcpdump src port 1025   // 源端口号为1025
    tcpdump src port 1025 and tcp
    tcpdump portrange 21-23
    tcpdump less 32   //只看到包低于或高于一定的大小
    tcpdump > 32

***2.与或非:***

1.  and 或者 &&, 在使用 && 的时候，要用单引号或者双引号包住表达式 `tcpdump -nvvv -i any -c 3 'port 22 && port 60738'`
2.  or 或者 \|\|
3.  not 或者 !

eg:

    tcpdump -nnvvS tcp and src 10.5.2.3 and dst port 3389

#### 查看指定的包类型

你还可以根据包里面的某些字段来组合各种条件指定你要关注的包。这个功能在你想要查看SYNs和RSTs非常有用。

eg:

    tcpdump 'tcp[13] & 32 != 0'  //查看所有的紧急包(URG包)
    tcpdump 'tcp[13] & 16 != 0'  //查看所有的确认包(ACK包)
    tcpdump 'tcp[13] & 8 != 0'  // 查看所有的PSH包
    tcpdump 'tcp[13] & 4 != 0'  //查看所有的RST包
    tcpdump 'tcp[13] & 2 != 0'  //查看所有的SYN包
    tcpdump 'tcp[13] & 1 != 0'  //查看所有的FIN包
    tcpdump 'tcp[13] = 18'  //查看所有的SYN-ACK包

ps:
只有PSH, RST, SYN 和 FIN 标志显示在tcpdump的标志域输出中。URG和ACK也会被显示，但显示在其它的地方而不是在标志位中。

#### 报文详解

\[S\] - SYN (开始连接)
\[.\] - 没有标记
\[P\] - PSH(数据推送)
\[F\] - FIN(结束连接)
\[R\] - RST(重启连接)
todo

### dig

![dig](/images/network/dig.png)

A记录和CNAME

* A记录：把一个域名解析到一个IP地址（Address，特制数字IP地址）
* CNAME记录：把域名解析到另外一个域名。
* SREVER: dns服务器+端口（53）

其功能是差不多，CNAME将几个主机名指向一个别名，其实跟指向IP地址是一样的，因为这个别名也要做一个A记录的。但是使用CNAME记录可以很方便地变更IP地址。

eg: 如果一台服务器有100个网站，他们都做了别名，该台服务器变更IP时，只需要变更别名的A记录就可以了。

    aa.bb.com  in cname xx.cname.com
    cc.dd.com  in cname xx.cname.com
    ee.ff.com  in cname xx.cname.com
    xx.cname.com in a 223.21.34.23

### ifconfig


### 参考

[tcp维基百科]<https://en.wikipedia.org/wiki/Transmission_Control_Protocol>

[tcpdump1]<http://blog.jobbole.com/91631/>

[tcpdump入门教程]<http://mp.weixin.qq.com/s?src=3&timestamp=1461728725&ver=1&signature=b1BrTvreiCRq8cd5MvCp4rtefuzjbTgTPSi6yhsoUR6EkFsiRgzP2q5MCoMjZEdzr1WdSiM5sg46uhXG79868TYLLq5-ztt6GjgP62dG2lBmSSmPINxVRJusc-VWBlsePOH4vhRarvrPHk0YXBR8sQ==>

[Linux 命令神器：lsof 入门]<http://mp.weixin.qq.com/s?src=3&timestamp=1461728725&ver=1&signature=YhWMmVMFsm8GFWX8kDhO6q4vtUEJTeXEicd6bs41iWlRPRfn-h2KAkOc-TNnBXAhPmywNtHhPRsmLGGxrkio*3gW9gBGByCuOLmEb1to75iv9O10EQ31gqiigNFJthDYS-40WA*m29-jAJwyF8aqOQ==>

[A记录和CName]<http://blog.xieyc.com/differences-between-a-record-and-cname-record/>

[understanding-the-dig-command]<https://mediatemple.net/community/products/dv/204644130/understanding-the-dig-command>
