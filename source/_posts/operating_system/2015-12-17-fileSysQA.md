---
layout: post
title: 文件系统x问
categories: os
tags: inode vnode dentry super_block file_system
---

*   [空文件占空间么，占的话占多少？](#empty_file)
*   [空目录占空间么，占的话占多少？](#empty_directory)
*   [对于空目录，inode存放文件的元信息，那么4096是干什么的呢？](#directory_4096)
*   [目录能保存的文件个数是否跟文件名长度相关？](#directory_and_file)
*   [touch只包含一个空格的文件，为什么占用了我4K的空间？](#empty_file_4k)
*   [ll第四列的总和（除去..的）与du -h的区别在哪？](#ll_and_du)


<http://djt.qq.com/article/view/620>基本上是照着这里走了一遍

<h3 id="empty_file">空文件占空间么，占的话占多少</h3>

    lcj@lcj:~$ touch empty.file; ll | grep empty.file
    -rw-rw-r--  1 lcj  lcj      0 12月 22 17:52 empty.file

好奇怪,空文件占0?，那操作系统是怎么知道空文件在哪的呢?必须肯定以及确定有地方放着其元信息．

`df -i` (查看inode节点使用情况)，然后重新touch一个新文件,然后再df -i下，看下used的数量，然后你会发现used的加了１

    df -i | grep /dev/sda       
    
//找你要查看的磁盘分区

    lcj@lcj:~/test/empty.dir$ sudo fdisk -l //找你要查看的磁盘分区

    Disk /dev/sda: 256.1 GB, 256060514304 bytes
    255 heads, 63 sectors/track, 31130 cylinders, total 500118192 sectors
    Units = 扇区 of 1 * 512 = 512 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x000a89d8

       设备 启动      起点          终点     块数   Id  系统
    /dev/sda1   *        2048   483534847   241766400   83  Linux
    /dev/sda2       483536894   500117503     8290305    5  扩展
    /dev/sda5       483536896   500117503     8290304   82  Linux 交换 / Solaris

//　查看对应磁盘分区inode的size

    lcj@lcj:~/test/empty.dir$ sudo dumpe2fs -h /dev/sda1 | grep -i 'inode.*size'
    dumpe2fs 1.42.9 (4-Feb-2014)
    Filesystem features:      has_journal ext_attr resize_inode dir_index filetype needs_recovery extent flex_bg sparse_super large_file huge_file uninit_bg dir_nlink extra_isize
    Inode size:	          256

那么为什么ll显示的文件大小为0呢？ll不带元信息呗,只是文件大小!

结论： ***空文件的大小至少为inode的size大小，ps: inode size 是在格式化的时候设定的*** 　　　　　　

<h3 id="empty_directory">空目录占空间么，占的话占多少</h3>

    lcj@lcj:~$ mkdir empty.dir;ll | grep empty.dir
    drwxrwxr-x  2 lcj  lcj   4096 12月 22 18:00 empty.dir/
空目录占4096,怎么不是0，也不是265?4096是什么玩意？按照上述操作，发现inodes　used的数量也加１了

//查看对应磁盘分区block的大小

    sudo dumpe2fs -h /dev/sda1 | grep -i 'block.*size'
结论： ***空目录的大小至少为block的size＋inode的size，同样block的size也是在文件系统格式化的时候设定的***

<h3 id="directory_4096">inode存放文件的元信息，那么这4096是干什么的呢？</h3>
我们做个猜想，存放文件夹与其中文件的关联信息．

    lcj@lcj:~/test/empty.dir/empty.fir$ du -h;
    4.0K	.
    lcj@lcj:~/test/empty.dir/empty.fir$ touch empty.file1 empty.file2;du -h
    4.0K	.

没啥变化，我touch1000个文件呢？脚本如下：

    #!/bin/bash
    for i in {1..1000}
    do
       touch empty.dir/$i;
    done
du -h 就是128K
结论： ***空目录的4096是是用来存目录与文件夹的关系的***

<h3 id="directory_and_file">文件名长度是否也跟目录能保存的文件个数相关？</h3>

当两个空文件时，du -h仍为4k，但是1000个文件时，就变了(如上图)，那么文件名长度是否也跟目录能保存的文件个数相关？
将文件名扩展了20多倍，用新的脚本重试上述步骤，然后就128K了，原来的24K.

结论： ***目录block中能保存的文件名个数和文件名的长度有关，实际上linux操作系统文件名最长为256bytes（你可以试试`touch 1111...1111`）***
所以当你的文件数量相当大的时候，你就要考虑你的文件名是否导致你的目录block占用太多．

<h3 id="empty_file_4k">touch只包含一个空格的文件，为什么占用了我4K的空间？</h3>

    lcj@lcj:~/test/empty.dir$ stat empty.file
     文件："empty.file"
     大小：2         	块：8          IO 块：4096   普通文件
    设备：801h/2049d	Inode：6029317     硬链接：1
    权限：(0664/-rw-rw-r--)  Uid：( 1000/     lcj)   Gid：( 1000/     lcj)
    最近访问：2015-12-17 22:17:01.927402539 +0800
    最近更改：2015-12-17 22:17:01.927402539 +0800
    最近改动：2015-12-17 22:17:01.927402539 +0800
    创建时间：-
结论： ***操作系统最小的分配单位就是block,就是上述的IO块，block和inode的size都是在分区时指定的***

<h3 id="ll_and_du">ll第四列的总和（除去.和..）与du -h的区别在哪？</h3>
结论： ***ll显示的是文件内容占用的实际字节数,du显示的占用的`block（IO块）大小* 4`***　
.与..存放的是目录与文件的关联关系，其数据结构todo