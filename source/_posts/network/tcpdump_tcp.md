---
layout: post
title: 通过tcpdump学习网络协议
date: 2016-05-07
categories: network
tag:
    - tcpdump
    - tcp
    - mtu
    - tos
    - segment
    - packet
    - fragment
---

在网络osi的七层模型中,TCP位于第四层-Transport(传输)层,IP在第三层—Network层，ARP在第二层—Data Link层，
在第二层上的数据，我们叫`Frame`，在第三层上的数据叫`Packet&datagram`，第四层的数据叫`Segment`

关于这几个术语,[wikipedia是这么说的](https://en.wikipedia.org/wiki/Transmission_Control_Protocol#TCP_segment_structure):

The term TCP packet appears in both informal and formal usage, whereas in more precise terminology ***segment*** refers to the TCP protocol data unit (PDU), ***datagram**, and ***frame*** to the data link layer PDU

大家意见是不统一的,我们这就frame,packet,segment来分别标识链路层,网络层和传输层

### ip packet header

![IPv4 header](/images/network/ip_header.png)

1.  `Version`:前四个字节为版本号: 0100  IPv4 , 0110 IPv6
2.  `Internet Header Length`: IHL ,四个字节, 它是32bits字的数量(32 * IHL),最小为5(0101),最大为15(1111),所以ip头的长度\[5\*32=160bits=20bytes,15\*32=480bits=60bytes\]
3.  `Differentiated Services Code Point && Explicit Congestion Notification`:  DSCP && ECN 包的优先级??
4.  `Total Length`: 16bits 定义了packet的大小,包括头(header)和数据(data),以字节为单位,最小20个字节,最大65535字节
5.  `Identification`: id标识
6.  `Flags`: 第一位reserved,必须为0;第二位 DF(Don't Fragment),DF位设为1时表明路由器不能对上层数据包分片。
如果一个上层数据包无法在不分片的情况下进行转发，则路由器会丢弃该上层数据包并返回一个错误信息;
第三位 MF(More Fragments)当路由器对一个上层数据包分片，则路由器会在除了最后一个分片的IP包的包头中将MF位设为1。
7.  `Fragment Offset`: 片偏移,长度13比特。表示该IP包在该组分片包中位置，接收端靠此来组装还原IP包.如果packet的大小大于MTU(maximum transmission unit 最大传输单元),就要进行[分片](#fragmentation).
8.  `Time To Live`: TTL  长度8比特。当IP包进行传送时，先会对该字段赋予某个特定的值。当IP包经过每一个沿途的路由器的时候，每个沿途的路由器会将IP包的TTL值减少1。如果TTL减少为0，则该IP包会被丢弃,通常会给发送者发送一个ICMP超时消息。
这个字段可以防止由于路由环路而导致IP包在网络中不停被转发
9.  `Protocol`: 协议 0x01 ICMP, 0x06 TCP, 0x11 UDP
10. `Header Checksum`: 16位,用来做IP头部的正确性检测，但不包含数据部分。因为每个路由器要改变TTL的值,所以路由器会为每个通过的数据包重新计算这个值。校验不通过,会丢到该包.
11. `Source address`: 32位,源ip
12. `Destination address`: 32位,目的ip
13. `Options`: 松散源路由(给出一连串路由器接口的IP地址,ip包必须沿着这些Ip地址传送,但允许在相继的两个IP地址之间跳过多个路由器);严格源路由;路由记录;时间戳
14. `Padding`: 补齐,ip header必须为32bits的倍数

#### 分片

网络协议使得网络可以相互的通信.这种设计为多种不同的物理设备提供了便利,它是独立于它所依赖的链路层
的.不同硬件的网络设备通常具备不同的传输速度,同时也往往有不同的最大传输单元.当一个网络向一个具备较小MTU的网络
传送数据报时,它可能需要对数据报进行分片.在IPv4中,这个功能被放在了网络层,由IPv4的路由器来实现.而在IPv6中,不允许路由器进行分片.主机在发送数据报之前必须能判定传输路径上的MTU.

当一个路由器接收到一个packet时,它检查目的地址,判定即将使用的接口和该接口的MTU.如果Packet的大小大于MTU,并且DF字段被设置为0,路由器就会将packet进行分片
路由器将packet分为多片,每片的大小最大为MTU - ip头的大小(20-60 bytes).每个fragment packet有以下变化:

1.  total length是fragment的size
2.  MF 标志被设为1(除了最后一个fragment,它被设为0)
3.  fragment offset被设置,它是以8byte来度量的
4.  header checksum被重新计算

eg: 如果MTU为1500 bytes,ip头大小为20,fragment的偏移将是(1500 - 20 ) / 8 = 185的倍数

如果一个传输层的segment的大小为4500bytes,ip头为20bytes,所以ip packet的大小为4500.当这个包经过一个MTU为2500的连接时,它会变成如下两片:

|Fragment|Total bytes|Header bytes|Data bytes|MF|Fragment offset(8-byte blocks)|
|-|-|-|-|-|-|
|1|2500|20|2480|1|0|
|2|2040|20|2020|0|310|

### tcp segment header

![tcp header](/images/network/tcp_header.png)

1.  `Source port`: 16位,源端口号
2.  `Destination port`: 16位,目的端口号
3.  `Sequence number`:32位 序列号
4.  `Acknowledgment number`:  32位    
5.  `Data offset`: 4位TCP header的size(20-60)bytes
6.  `Reserved`: 3位
7.  `NS`: 1位
8.  `CWR`: 1位
9.   `ECE`: 1位
10. `URG`: 1位 表明Urgent pointer是有意义的
11. `ACK`: 1位 表明Acknowledgment 域是有意义的,在初始化的SYN包后的所有的包的这个位都应该被置为1
12. `PSH`: 1位 请求推送数据给接收的应用
13. `RST`: 1位 重置连接
14. `SYN`: 1位 同步序列号.仅当client和server两端第一次发送包时这个位被置为1
15. `FIN`: 1位 不再需要数据了
16. `Window size`: 16位 接收的窗口大小
17. `Checksum`: 16位 校验和
18. `Urgent pointer`:  16位 如果URG位被置为1,这表明上一个urgent数据byte相对于它的序列号的偏移
19. `Options`:
20. `Padding`: 补0,

### practice

好了,有了上述基础,我们来实操下吧.
dump下线上一个包,[tcpdump入门](/2015/03/20/network#tcpdump)

    [chuangjian.li@l-xxxx.f.cn1 ~]$ sudo tcpdump -nnvvX host  218.80.232.44 -c 3
    tcpdump: listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
    20:17:08.896034 IP (tos 0x28, ttl 49, id 16341, offset 0, flags [none], proto TCP (6), length 1400)
        218.80.232.44.80 > 10.86.212.176.33314: Flags [.], cksum 0x195c (correct), seq 1024948164:1024949512, ack 2128146148, win 65535, options [nop,nop,TS val 2979322617 ecr 968275531], length 1348
        0x0000:  4528 0578 3fd5 0000 3106 a2ff da50 e82c  E(.x?...1....P.,
        0x0010:  0a56 d4b0 0050 8222 3d17 77c4 7ed8 eee4  .V...P."=.w.~...
        0x0020:  8010 ffff 195c 0000 0101 080a b194 daf9  .....\..........
        0x0030:  39b6 b64b 4854 5450 2f31 2e31 2032 3030  9..KHTTP/1.1.200
        0x0040:  204f 4b0d 0a44 6174 653a 204d 6f6e 2c20  .OK..Date:.Mon,.
        0x0050:  3039 204d 6179 2032 3031 3620 3132 3a31  09.May.2016.12:1
        0x0060:  343a 3339 2047 4d54 0d0a 5365 7276 6572  4:39.GMT..Server
        0x0070:  3a20 4170 6163 6865 0d0a 582d 4672 616d  :.Apache..X-Fram
        0x0080:  652d 4f70 7469 6f6e 733a 2053 414d 454f  e-Options:.SAMEO
        0x0090:  5249 4749 4e0d 0a56 6172 793a 2041 6363  RIGIN..Vary:.Acc
        0x00a0:  6570 742d 456e 636f 6469 6e67 0d0a 5472  ept-Encoding..Tr
        0x00b0:  616e 7366 6572 2d45 6e63 6f64 696e 673a  ansfer-Encoding:
        0x00c0:  2063 6875 6e6b 6564 0d0a 436f 6e74 656e  .chunked..Conten
        0x00d0:  742d 5479 7065 3a20 7465 7874 2f78 6d6c  t-Type:.text/xml
        0x00e0:  3b63 6861 7273 6574 3d55 5446 2d38 0d0a  ;charset=UTF-8..
        0x00f0:  436f 6e6e 6563 7469 6f6e 3a20 4b65 6570  Connection:.Keep
        0x0100:  2d41 6c69 7665 0d0a 0d0a 3166 6638 0d0a  -Alive....1ff8..
        ....

-nn 不解析域名和端口名 -vv 显示多的信息 -X 以ascii显示报文信息,不显示frame头,所以上述的就是一个ip packet.

一个字节一个字节来:
***首先IP头***

4: Version IPv4 版本4

5: IHL 5 * 4 = 20 bytes ip头大小为20bytes

28: tos 为0x28 ,`tos`

0578: 十进制1400 ip packet的长度为1400bytes,`length`

3fd5: 十进制16341 `id`

0(3bits): DF 与MF都为0 ,说明发生了发片,且不是最后一片

000(13bits): Fragment offset,片偏移为0,分片后的第一片

31: 十进制49 `ttl`

06: 十进制6,对应tcp协议 `Protocol`

a2ff: `Header Checksum`

da50e82c: 0xda十进制218, 0x50十进制80,0xe8十进制232,0x2c十进制44 ,对应源ip218.80.232.44`Source address`

0a56d4b0: 同上`Destination address`

***刚好20个字节,接下来TCP头***

0050: 十进制80,源端口号`Source port`

8222: 十进制33314,目的端口号`Destination port`

3d17 77c4: 照不上

7ed8eee4:十进制2128146148,`Acknowledgment number`

8: 8 * 4= 32 tcp header的大小为32`Data offset`

0(3bits): 000`Reserved`

010(9bits): 000010000 `ACK`为1 ??

ffff: 65535 窗口大小`Window size`

195c: `Checksum`

0000:`Urgent pointer`

这才20bytes了,还有32-20 =12 bytes 待研究 `0101 080a b194 daf9 39b6 b64b`

***再接下来就是HTTP报文了***

4854 5450 2f31 2e31 2032 3030 ....

### 参考

[wiki百科ip]<https://en.wikipedia.org/wiki/IPv4#Header>

[wiki百科tcp]<https://en.wikipedia.org/wiki/Transmission_Control_Protocol>

[Packet Fragmentation]<http://www.tech-faq.com/packet-fragmentation.html>

[DSCP]<https://en.wikipedia.org/wiki/Differentiated_services>

[TOS]<https://en.wikipedia.org/wiki/Type_of_service>

[wiki校验和]<https://en.wikipedia.org/wiki/IPv4_header_checksum>

[IP头结构详解]<http://blog.csdn.net/achejq/article/details/7040687>

[tcp的那些事儿[上]]<http://coolshell.cn/articles/11564.html>

[tcp的那些事儿[下]]<http://coolshell.cn/articles/11609.html>
