---
layout: post
title: fiddler使用
date: 2015-12-28
categories: network
tags:
    - fiddler
---

### ubuntu使用

#### 方法一：linux 直接安装

1.  sudo apt-get install mono-complete

2.  下载fiddler　http://fiddler.wikidot.com/mono

3.  解压，进入目录，执行mono Fiddler.exe

#### 方法二：安装windows虚拟机，桥接模式使用

1.  设置虚机 安装虚拟机，网络模式设为桥接模式 　

    VM-settings-Network Adapter-Network Connection-Bridged

2.  设置fiddler　

    Tools-Fiddler Options-Connections-Allow remote computers to connect

3.  设置ubuntu

    系统设置－网络－网络代理－方法－手动－添加虚机windows的 ip和端口，端口默认8888

#### https

1. Tools-Fiddler Options-HTTPS

2. 勾选decrypt https traffic  和 ingore server certificate errors

3. actions-export fiddler root certificate to desktop

4. yes ok

5. chrome 可以将这个证书加入信任列表　设置-HTTPS/SSL-管理证书-授权中心-导入

idea debug使用
设置ip和端口为代理ip 和代理端口8888即可。

### mac使用

1. 下载安卓模拟器

2. 使用charles或者wireshark抓包，选择http

### 手机抓包
电脑创建wifi热点,让手机的流量都都电脑

1.  windows创建的就不说了　
2.  [ubuntu创建wifi热点](http://ubuntuhandbook.org/index.php/2014/09/3-ways-create-wifi-hotspot-ubuntu/)

    wifi tab下：

    *   wifi模式选择infrastructure(架构)
    *   MAC address 选择下拉的那个

    WIFI security　tab下：

    *   选择WPA&WPAx Personal

    IPv4 Setting tab下：

    *   Shared to other computers

    编辑 /etc/NetworkManager/system-connections/xxx_wifi_name
    将mode=infrastructure 改成　mode=ap，保存，连接

    然后手机上链接wifi热点，设置代理ip为fiddler上那个图标显示的ip port就行．(我的魅族是长按对应wifi来设置代理)



参考

[1]<http://www.cnblogs.com/TankXiao/archive/2012/02/06/2337728.html>

[2]<http://www.imooc.com/learn/37>
