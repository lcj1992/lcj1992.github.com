---
layout: post
title: 同时使用内网和翻墙
categories: tool
tags: great_wall
---

goagent：（好像确实不是很稳定）

http://maolihui.com/goagent-detail.html

vpn：

材料准备
购买vpn,我用的云梯

连接vpn之后上不了内网，同时上内网和翻墙

1.先连接到内网

2.执行下边脚本

3.设置内网不走vpn,走默认网关

killGW.sh

    #!/bin/bash
    # mac 用这个
    # OLDGW=`route -n get default | grep gateway | awk  '{print $2}'`
    OLDGW=`ip route show | grep '^default' | sed -e 's/default via \([^ ]*\).*/\1/'`
    sudo route add -net xx.xx.0.0 netmask 255.255.0.0 gw $OLDGW
    sudo route add -net xxx.xxx.0.0 netmask 255.255.0.0 gw $OLDGW

![原理](/images/tool/yuanli.png)

4.翻墙

google：http://fxck.it/post/57501298563