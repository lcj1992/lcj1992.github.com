---
layout: post
title: 磁盘满了
date: 2015-12-17
categories:
    - performance
---

### 一般解决办法

1.先查看满了的是哪个磁盘

*   df  　报告文件系统磁盘使用率
`df -h 　打印m,k,g等单位`
`df -i 　打印inodes信息`
2.找到了是哪个磁盘，看看是这个磁盘的哪些文件

*   du   统计每个文件的磁盘使用，目录的话会递归
`du -m  以兆为单位打印`
`du -m | sort -nr | uniq | head -10  打印占用前十的文件`
df发现所有磁盘空间都足足的

#### inode不够用了

`df -i 查看磁盘inode使用情况`
//可以使用这个脚本来创建大量空文件,加以验证

        #!/bin/bash
        for i in {10000..60000}
        do
            touch `date`+$i;
        done

![inode耗尽](/images/performance/df-i.png)

#### 删除正在被其他进程占用的文件

删除的只是文件名，并没有删除其inode，所以占用空间，但是du看不到。
