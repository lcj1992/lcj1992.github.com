---
layout: post
title: 通过clion查看mysql-server源码
date: 2017-10-10
categories: db
tags:
    - mysql-server
    - source
---

### 下载编译


* 下载源码

```
git clone https://github.com/mysql/mysql-server.git
```

* 生成make脚本

```
cmake \
-DCMAKE_INSTALL_PREFIX=/Users/lichuangjian/soft/mysql \
-DMYSQL_DATADIR=/Users/lichuangjian/soft/mysql/data \
-DSYSCONFDIR=/Users/lichuangjian/soft/mysql/etc \
-DMYSQL_UNIX_ADDR=/Users/lichuangjian/soft/mysql/mysql.sock \
-DWITH_DEBUG=1  \
-DDOWNLOAD_BOOST=1 -DWITH_BOOST=/Users/lichuangjian/soft/mysql/ -DDOWNLOAD_BOOST_TIMEOUT=60000
```

* 编译

`make -j 4`

* 连接打包

`make install -j 4`

* 初始化

`mysqld --initialize-insecure --user=root --datadir=/Users/lichuangjian/soft/mysql/data`


### 导入clion

cmake配置
要和之前的一样
![cmake配置](/images/database/clion_config_1.png)
![cmake配置](/images/database/clion_config_2.png)


### 参考

[怎么样使用CLion调试分析MySQL Server](https://zhidao.baidu.com/question/1707589153063680500.html)

[怎么样使用CLion调试分析MySQL Server](https://my.oschina.net/u/222608/blog/1511382)

http://blog.csdn.net/king_wangheng/article/details/7592277

http://www.uml.org.cn/sjjm/201107145.asp

http://www.cnblogs.com/zhoujinyi/archive/2013/04/16/3016223.html
