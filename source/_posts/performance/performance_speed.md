---
layout: post
title: 一些速度
date: 2015-12-27
categories: performance
---

#### 一些速度

|硬件|　速度|
|-|-|
|L1 cache reference 读取cpu的一级缓存	|0.5ns
|Branch mispredict 转移，分支预测	|5ns
|L2 cache reference 读取cpu的二级缓存	|7ns      (14 × L1cache)
|Mutex lock/unlock 互斥锁/解锁	|25ns
|Main memory reference 读取内存数据|	100ns  (20 × L2 cache，200 × L1 cache)
|Compress 1k bytes with zippy 1k字节压缩	|3000ns
|Send 1k bytes over 1 Gbps network 在1Gbps的网络上发送1k字节ps:  1Gbps （带宽）　一秒发送１Ｇbits（不是bytes）	|10,000ns   0.01ms
|Read 4k randomly from SSD	|150,000ns  0.15ms
|Read 1MB sequentially from memory  从内存顺序读取1MB	|250,000ns  0.25ms
|Round trip within same datacenter 从一个数据中心往返一次，ping 一下	|500,000ns
|Disk seek  磁盘搜索	|10,000,000ns  20 x datacenter roundtrip
|Read 1MB sequentially from network 从网络上顺序读取1兆数据	|10,000,000ns
|Read 1MB sequentially from disk  从磁盘读取1兆	|30,000,000ns
|Send packet CA ->Netherlands -> CA 一个包的一次远程访问	|150,000,000ns

根据上述，算一下一秒读取的字节数

*   1Gbps的带宽　128M
*   内存中顺序读取　4G
*   从ssd中随机读取　27M　顺序读取　　200+M
*   从网络顺序读取　100M
*   从磁盘顺序读取　33M
