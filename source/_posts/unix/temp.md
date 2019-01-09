---
layout: post
title: 尚未安家的
date: 2015-12-27
categories: linux
tags: date id scp debug clion ctags cscope whois which whereis whatis ldd screen sublime
---

1.id
查看自己的用户id，组id

    lcj@lcj:~$ id
    uid=1000(lcj) gid=1000(lcj) 组=1000(lcj),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),108(lpadmin),124(sambashare),129(docker)

2.linux机器间文件copy
scp 本地文件 远程ip:远程文件

scp  -r 本地目录 远程ip:远程目录

前提是机器上有你的公钥

    scp chuangjian.li@xx:/home/lcj/xx ~/

字符转码

    iconv -f windows-1252 -t utf-8 infile > outfile

/dev/null

    cat aa > /dev/null 2>&1

/dev/null 是个黑洞,2>&1将标准错误重定向到标准输出,
0 标准输入

1 标准输出

2 标准错误

http://www.xaprb.com/blog/2006/06/06/what-does-devnull-21-mean/

远端执行命令

    ssh chuangjian.li@xx  'command1 ;command 2; command3'

ssh优雅地退出连接

`~.`

<http://www.vpsee.com/2013/08/how-to-kill-an-unresponsive-ssh-connection/>

http://www.laoono.com/s-db/26.html

用户

    cat /etc/passwd
    lcj:x:1000:1000:lcj,,,:/home/lcj:/bin/bash

*    x         用户密码，不显示，加密后的密码保存在/etc/shadow
*    1000      用户id
*    1000      组id
*    lcj,,,    用户描述，全名，家名，工作号码，家庭号码等。
*    /home/lcj 用户的家目
*    /bin/bash 用户登陆的shell

环境变量

/etc/profile : 在登录时,操作系统定制用户环境时使用的第一个文件 ,此文件为系统的每个用户设置环境信息,当用户第一次登录时,该文件被执行。
/etc/environment : 在登录时操作系统使用的第二个文件, 系统在读取你自己的profile前,设置环境文件的环境变量。
~/.profile :  在登录时用到的第三个文件 是.profile文件,每个用户都可使用该文件输入专用于自己使用的shell信息,当用户登录时,该文件仅仅执行一次!默认情况下,他设置一些环境变量,执行用户的.bashrc文件。
/etc/bash.bashrc : 为每一个运行bash shell的用户执行此文件.当bash shell被打开时,该文件被读取.
~/.bashrc : 该文件包含专用于你的bash shell的bash信息,当登录时以及每次打开新的shell时,该该文件被读取。

（用户独有的配置，好多都是其家目录下的    .文件）
http://vic.gedris.org/Manual-ShellIntro/1.2/ShellIntro.pdf

### clion注册机
CLion注册时选择“License server”输入<http://idea.lanyus.com>点击“OK”，不用打补丁直接激活


### tomcat远程调试

１.服务器设置

    -Xdebug -Xnoagent -Djava.compiler=NONE -Xrunjdwp:transport=dt_socket,address=8419,server=y,suspend=n
    socat TCP4-LISTEN:50001,fork,range=localIp/32 TCP4:127.0.0.1:50002

2、在idea中设置远程调试的Host ip为serverIp, 端口为serverPort

idea:run->Edit Configurations->添加remote->修改ip地址+端口号

使用
1.调试  2.debug状态下改值跑case      F2，F9

注：内网ip隔几天会换的

date=`date +%Y-%m-%d`

date -d@1234567 时间戳转化成时间, 不带毫秒. java new Date(longVal) longVal需要加000,java中带毫秒.

使用ldd来查看某个软件所依赖的共享库,如查看nginx,看那个no found 去/usr/lib下去找看是否有类似的

    ldd `which nginx`

cd - 　回到上一个工作路径

alias xx='xxCommand'

mac软件版本太低，又卸载不了，如何使用brew安装新的版本并覆盖老版本？
PATH中将/usr/local/bin/  放在/usr/bin之前

通过使用 <(some command) 可以将输出视为文件。例如，对比本地文件 /etc/hosts 和一个远程文件：

      diff /etc/hosts <(ssh somehost cat /etc/hosts)

HttpClient4.x可以自带维持会话功能，只要使用同一个HttpClient且未关闭连接，则可以使用相同会话来访问其他要求登录验证的服务

idea 类图插件 fn + delete 删除类

predicate 谓词：在计算机语言的环境下，谓词是指条件表达式的求值返回真或假的过程。

pstree 可以查看进程包含的线程数

eval "$(ssh-agent -s)"


shell sleep 1s



https://stackoverflow.com/questions/2983248/com-mysql-jdbc-exceptions-jdbc4-communicationsexception-communications-link-fai

###  参考

[screen]<http://www.ibm.com/developerworks/cn/linux/l-cn-screen/>

[Httpclient4.x使用cookie保持会话]<http://sb33060418.iteye.com/blog/2020007>
