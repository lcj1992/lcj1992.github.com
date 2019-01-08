---
layout: post
title: redis集群搭建
categories: soft
tags: redis cluster sentinel
---

    //下载redis源码
    git clone git@github.com:antirez/redis.git
    // 编译
    make noopt
    // 安装
    make install

    //redis-7000.conf 配置文件
    port 7000  
    daemonize yes  
    pidfile /var/run/redis-7000.pid  
    logfile "7000.log"  
    dbfilename "dump-7000.rdb"  
    appendonly yes  
    appendfilename "appendonly-7000.aof"  
    dir "/data/redis"

    // 同样的编辑redis-7001.conf 和redis-7002.conf
    sed 's/7000/7001/g' redis-7000.conf >>redis-7001.conf
    sed 's/7000/7002/g' redis-7000.conf >>redis-7002.conf

    // 设置主从关系
    echo "slaveof 192.168.1.108 7000" >>redis-7001.conf
    echo "slaveof 192.168.1.108 7000" >>redis-7002.conf

    // 启动redis
    redis-server /cluster/redis-7000.conf
    redis-server /cluster/redis/redis-7002.conf
    redis-server /cluster/redis/redis-7001.conf
   
    // 编辑redis-sentinel-26379.conf
    port 26379  
    daemonize yes  
    pidfile /var/run/redis-26379.pid
    logfile "26379.log"
    dir /data/redis
    sentinel monitor mymaster 127.0.0.1 7000 2
    sentinel down-after-milliseconds mymaster 30000
    sentinel parallel-syncs mymaster 1
    sentinel failover-timeout mymaster 180000
    
    // 同样的编辑redis-sentinel-26380.conf和redis-sentienl-26381.conf
    sed 's/26379/26380/g' redis-sentinel-26379.conf >> redis-sentinel-26380.conf
    sed 's/26379/26381/g' redis-sentinel-26379.conf >> redis-sentinel-26381.conf

    // 启动哨兵
    sed 's/26379/26380/g' redis-sentinel-26379.conf >> redis-sentinel-26380.conf
    sed 's/26379/26381/g' redis-sentinel-26379.conf >> redis-sentinel-26381.conf

    
### 参考 

[Redis 自动故障转移sentinel（哨兵）实践]<http://lampblog.org/1840.html>
