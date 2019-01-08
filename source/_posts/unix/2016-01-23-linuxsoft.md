---
layout: post
title: linux软件安装
categories: unix
---

`apt-get`,`yum` ,`dnf`,`pacman`,`pip`,`brew`，`port`

#### ubuntu

    sudo apt-get install xxx //安装
    sudo apt-get remove xxx　//卸载

#### centos

    sudo yum install xxx
    sudo yum remove xxx

#### mac

mac的包管理工具默认是不带的，需要自己安装。

>  1.[homebrew](http://brew.sh/)，安装的软件在/usr/local/bin

>  2.[macports](https://www.macports.org/install.php),直接dmg安装就行，安装的软件在/opt/local/bin


    brew install xxx
    brew remove xxx
    port install xxx
    port remove xxx

#### 源码安装的

    ./configure //检查依赖关系是否满足
    make    //编译
    make install // 安装
    make　uninstall //卸载 如果软件源码删了，那么就不能了．
