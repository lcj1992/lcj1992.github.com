---
layout: post
title: uuid(抄)
date: 2016-01-08
categories: java_web
tags:
    - uuid
    - guid
---

`uuid`:通用唯一识别码是一种软件建构的标准，亦为自由软件基金会组织在分散式计算环境领域的一部份。 UUID的目的，是让分散式系统中的所有元素，都能有唯一的辨识信息，而不需要通过中央控制端来做辨识信息的指定。如此一来，每个人都可以创建不与其它人冲突的UUID。

### uuid版本

uuid具有多个版本，每个版本的算法也不同，应用范围也不同

#### 版本1

基于时间的UUID通过计算当前时间戳、随机数和机器MAC地址得到。由于在算法中使用了MAC地址，这个版本的UUID可以保证在全球范围的唯一性。但与此同时，使用MAC地址会带来安全性问题，这就是这个版本UUID受到批评的地方。如果应用只是在局域网中使用，也可以使用退化的算法，以IP地址来代替MAC地址－－Java的UUID往往是这样实现的（当然也考虑了获取MAC的难度）。

#### 版本2

DCE（Distributed Computing Environment）安全的UUID和基于时间的UUID算法相同，但会把时间戳的前4位置换为POSIX的UID或GID。这个版本的UUID在实际中较少用到。

#### 版本3

基于名字的UUID通过计算名字和名字空间的MD5散列值得到。这个版本的UUID保证了：相同名字空间中不同名字生成的UUID的唯一性；不同名字空间中的UUID的唯一性；相同名字空间中相同名字的UUID重复生成是相同的。

#### 版本4

根据随机数，或者伪随机数生成UUID。这种UUID产生重复的概率是可以计算出来的，但随机的东西就像是买彩票：你指望它发财是不可能的，但狗屎运通常会在不经意中到来。

#### 版本5
和版本3的UUID算法类似，只是散列值计算使用SHA1（Secure Hash Algorithm 1）算法。

### uuid应用
从UUID的不同版本可以看出，Version 1/2适合应用于分布式计算环境下，具有高度的唯一性；Version 3/5适合于一定范围内名字唯一，且需要或可能会重复生成UUID的环境下；至于Version 4，我个人的建议是最好不用（虽然它是最简单最方便的）。

guid是微软对uuid标准的实现

[1]<http://www.blogjava.net/feelyou/archive/2008/10/14/234320.html>

[2]<https://en.wikipedia.org/wiki/Universally_unique_identifier>