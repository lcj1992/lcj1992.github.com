---
layout: post
title: 编译openJdk
categories: java
description: 编译openjdk
keywords: openjdk
---

下载openjdk

设置环境

    export LANG=C
    export ALT_BOOTDIR=/home/q/java/jdk1.7.0_71/      （机器上有jdk才能编译jdk，因为一部分是c代码一部分是java代码）    
    export ALLOW_DOWNLOADS=true  
    unset CLASSPATH  
    unset JAVA_HOME

检测

    make sanity

编译

    make clean
    make

出现的问题

1.  版本不对，出现找不到类的情况 adpterHandle
2.  test.gammma
3.  大于十年
我找到那个类GenerateCurrencyData.java，把10改成20了

编译后
编译后的jdk在build/j2sdk-image中





## mac ox10.11编译openjdk

    brew install mercurial
    brew install ccache
    brew install freetype
    mkdir openjdk9
    cd openjdk9
    hg clone http://hg.openjdk.java.net/jdk9/dev
    cd dev
    chmod u+x get_source.sh
    ./get_source.sh
    bash ./configure --enable-debug --with-target-bits=64 -with-freetype-include=/usr/local/Cellar/freetype/2.6.5/include/freetype2 -with-freetype-lib=/usr/local/Cellar/freetype/2.6.5/lib
