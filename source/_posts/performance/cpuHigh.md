---
layout: post
title: cpu高了
date: 2015-12-17
categories: performance
tags: cpu performance
---

### 案例一:死循环 

#### 测试代码

    public class HeapOom {
        static class OOMObject {
        }
        public static void main(String[] args) {
            List<OOMObject> list = new ArrayList<OOMObject>();
            while (true) {
                list.add(new OOMObject());
            }
        }
    }

#### 检测
1.top拿到cpu密集的java进程 24744，shift + H拿到cpu密集的线程 24745 `top -Hp 24744`,（mac似乎不支持查找线程）

![top](/images/performance/top.png)

2.`jstack -F 24744 > stack.log`

3.检测对应代码 

![cpu](/images/performance/cpu.png)

ps: printf "%0x\n" 24745找到nid为24745对应的线程。nid=24745

4.卧槽，死循环了。（当然这个还内存溢出）

### 案例x

### 查看最耗CPU的线程

    #!/bin/bash

    if [ $# -eq 0 ];then
        echo "please enter java pid"
        exit -1
    fi

    pid=$1
    jstack_cmd=""

    if [[ $JAVA_HOME != "" ]]; then
        jstack_cmd="$JAVA_HOME/bin/jstack"
    else
        r=`which jstack 2>/dev/null`
        if [[ $r != "" ]]; then
            jstack_cmd=$r
        else
            echo "can not find jstack"
            exit -2
        fi
    fi

    #line=`top -H  -o %CPU -b -n 1  -p $pid | sed '1,/^$/d' | grep -v $pid | awk 'NR==2'`

    line=`top -H -b -n 1 -p $pid | sed '1,/^$/d' | sed '1d;/^$/d' | grep -v $pid | sort -nrk9 | head -1`
    echo "$line" | awk '{print "tid: "$1," cpu: %"$9}'
    tid_0x=`printf "%0x" $(echo "$line" | awk '{print $1}')`
    #$jstack_cmd $pid | grep $tid_0x -A20 | sed -n '1,/^$/p'
    sudo -u tomcat $jstack_cmd $pid | grep $tid_0x -A20 | sed -n '1,/^$/p'

#### 参考

[1]<http://lovestblog.cn/blog/2016/03/31/cpu-thread/>

[2]<http://hongjiang.info/find-busiest-thread-of-java>
