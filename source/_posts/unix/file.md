---
layout: post
title: 文件相关命令
date: 2014-11-13
categories: unix
tags:
    - file
    - cd
    - ls
    - stat
    - chmod
    - chown
    - tree
    - z
    - find
    - locate
    - xargs
---

#### ls -l

    drwxr-xr-x 29 ownername  groupname  4096 11月 21 13:51 filename

*   文件类型
    	d 目录   
    	l 符号链接   
    	s 套接字文件   
    	b 块设备文件   
    	c 字符设备   
    	p 命名管道文件   
    	- 普通文件   
*   2-4位  文件属主拥有的权限  r 读 w 写 x 可执行
*   5-7位  同一组的 用户使用的权限
*   8-10位  其他用户所拥有的权限
*   29是该目录下含有的文件数(不包括.但是包括..)
*   ownername　　用户名
*   groupname   组名
*   4096　文件的大小
*   文件创建的日期
*   文件创建的时间
*   文件名

#### stat

显示文件的inode信息

    stat testfile
      文件："testfile"
      大小：2         	块：8          IO 块：4096   普通文件
    设备：801h/2049d	Inode：5903681     硬链接：2
    权限：(0664/-rw-rw-r--)  Uid：( 1000/     lcj)   Gid：( 1000/     lcj)
    最近访问：2015-07-04 18:57:47.828686719 +0800
    最近更改：2015-07-04 18:57:47.828686719 +0800
    最近改动：2015-07-04 19:01:05.780692592 +0800
    创建时间：-

      File: "test"
      Size: 3             Blocks: 8          IO Block: 4096   普通文件
    Device: fd01h/64769dInode: 926117      Links: 1
    Access: (0666/-rw-rw-rw-)  Uid: (1423194470/lichuangjian)   Gid: (1423194470/lichuangjian)
    Access: 2017-01-09 19:45:54.371609214 +0800
    Modify: 2017-01-09 19:45:54.371609214 +0800
    Change: 2017-01-09 19:46:29.493610603 +0800

大小： 文件大小(字节)

块： 扇区数（512字节为一扇区）

IO块：文件系统的逻辑块大小，一个文件最少也要占这么多个字节

设备：设备号

硬链接：多少个文件直接索引这个inode

权限： 三位三位的算 6 = 110 也就是rw- ，4 = 100也就是r--

uid：所有者id

gid：组id

最近访问(access)：表示我们最后一次访问文件的时间(可以是读也可以是写)
最近更改(modify): 表示我们最后一次修改文件的时间(内容变更)
最近改动(change): 表示我们最后一次对文件属性改变的时间(内容变更，权限，属性改变等)

eg:

1. 当我们仅仅只是读取文件时，access time改变，而modify，change不变
2. 当修改文件时，access，modify，change都会改变
3. 当修改文件属性时，change time改变，而modify和access不变。
4. ls -l 显示的是文件的modify时间

#### chmod

`chmod [who] operator [permission] filename`
who:  

*    u  文件属主
*    g  同一组的其他用户  
*    o  非同一组的用户  
*    a  所有用户  

operator：  

*    \+  加
*    \-  减
*    =  设定  

permission：    

*    r
*    w  
*    x  
*    s  文件属主和组setID  ?????
*    t  粘性位  该目录中的文件只有属主可以删除，即使某用户拥有和属主拥有一样的权限  
*    l  给文件加锁，使其他用户无法访问  

#### chown

 将file属主改为lcj  
`chown lcj file`

#### chgrp

 将file所属组改为hehe  
`chgrp hehe file`

#### ln

创建软链接
`ln -s`  
软链接和硬链接，后者直接索引文件索引节点，前者只是符号链接，相当于windows的快捷方式

![软硬链接](/images/unix/soft_hard_link.jpg)

#### locate

    locate xxx　定位文件
    find / -name xxx

#### find

查找某目录下某名字的文件

find xxxxPath   -name  filename

递归打印某目录下所有文件
find　xxxpath -print

#### tar

tar命令用来生成归档文件，以及将归档文件展开


|选项|作用|
|-|-|
|-c|create 生成新的包,如果原来存在,会先干掉之前的,重新生成一个|
|-r|也是个打包的命令,只不过如果存在包,不删除,只在包中追加文件,即使这个文件存在,还是会在加一份|
|-u|也是个打包命令,如果包中存在该文件,会比较更新时间,采用最新的版本的文件|
|-t|list,单纯地查看,不能与打包的,解包的混用|
|-x|解包|
|-z|gzip压缩,解压缩,效果还是挺明显的|
|-j|使用bzip2压缩,解压缩|
|-v|显示执行过程,类似于进度条的作用|
|-f|其后必须紧跟文件名|
|-p|使用原文件原有的属性(属性不会依据使用者而变)|
|--exclude File(可以是目录)|不打包File|

打包的有`-c`, `-r` ,`-u`;看包的有`-t`;解包的有`-x`,这五个选项都不能共存

eg:

    1.// 将aa bb cc 打包(-c)并压缩(-z),打包过程中显示执行过程(-v),打到aa.tar.gz这个包中(-f)
    tar -zvcf  aa.tar.gz   aa  bb cc

    2.// 将aa.tar.gz先解压,然后拆包,并显示执行过程
    tar -zxvf aa.tar.gz

    3.// 先解压,然后查看包中内容
    tar -ztvf aa.tar.gz

    4.// 不打包部分
    tar --exclude aa/xx  -zcvf  test.tar.gz aa  bb

#### cd

cd - 回到上一个工作目录

### z

1.  下载z.sh[z]<https://github.com/rupa/z>
2.  然后在.bashrc中指定z.sh的路径即可
3.  mac中如果使用了oh－my－zsh框架，在.zshrc中的plugins中添加z就可以了

### tree

    brew install tree
    tree -L 2  // 查看当前牧流，但只看2个层级

### xargs

1.  -n  一行输出多少个 `cat arg.txt | xargs -n2 cmdxx`
2.  -d  自定义分隔符  `cat arg.txt | xargs -dX cmdxx`
3.  -I  指定替换字符 `cat arg.txt | xargs -I fuck cmdxx arg1 fuck arg3 `
4.  -0  通常和`find`的-print0一块使用  `find . -type f -name "*.log" -print0 | xargs -0 rm -f`

[命令行的艺术](https://github.com/jlevy/the-art-of-command-line/blob/master/README-zh.md#命令行的艺术)

[2](http://man.linuxde.net/xargs)

[tar](http://blog.csdn.net/eroswang/article/details/5555415/)

[shell经典笔记](http://wangxinchun.iteye.com/blog/1872951)
