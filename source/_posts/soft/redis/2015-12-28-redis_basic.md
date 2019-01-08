---
layout: post
title: hello redis
categories: soft
tags: redis ds
---




redis是一个开源的内存数据结构存储，被用作数据库，缓存，以及消息中间件。它提供了诸如strings，hashes，lists，sets，带区间查询的sorted sets，bitmaps，hyperloglogs，以及带半径查询的geospatial indexes。redis内置了复制，lua 脚本，LRU逐出，事务以及不同级别的持久化，提供了带有哨兵和集群自动分区功能的redis集群。

### 介绍 

    brew install redis

或者源码安装

    wget http://124.202.164.14/files/50340000087E9816/download.redis.io/releases/redis-3.2.1.tar.gz
    tar -zxf redis-3.2.1.tar.gz
    cd redis-3.2.1
    make install

### 基本用法 

***首先明确一点这是一个key-value缓存系统，key是字符串，而这个value可以是多样的：string，map，list，set，sort set，bitmaps，hyperLogLogs***

    // 连接
    redis-cli  -h host -p port

    // 查看手册
    help set

    // 字符串
    set name lcj

    // hash
    hmset user:1 name fuck birth 18400101 phone 15945612345(对象,多个域)
    hset user:1 name fuck（一个域）
    hget user:1 name

    // 列表
    lpush lcj aa
    lrange a 0 10

    // 集合
    sadd lcj aa
    sadd lcj bb
    smembers lcj

    // 有序集合
        key     score       member
    zadd     9C.order  -235  235|sueccess
    zadd     9C.order - 234  234|fail
    zrange    9C.order 0 -1

    // 位图
    key   offset  0|1
    setbit active_user 001 1
    setbit active_user 002 0
    setbit active_user 003 1
    bitcount active_user
    getbit active 003

    // hyperLogLog
    PFADD ip '192.168.0.1'
    PFADD ip '192.168.0.1'
    PFCOUNT ip

除此之外还有发布订阅，事务的支持

    //一端
    subscribe heheda
    //另一端
    publish heheda helloIamHeheda

    //事务
    multi
    ...
    exec

### 参考 

[redis官网]<http://redis.io>
