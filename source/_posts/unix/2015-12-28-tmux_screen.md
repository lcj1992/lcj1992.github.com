---
layout: post
title: tmux & screen @Deprecated
categories: unix
tags: tmux screen
---

买了alfred和dash后，发现这篇文章完全可以废掉了,因为dash里有tmux和screen的手册，而且清晰简单，不像有些linux命令的手册似的，晦涩难懂。

之前错误的认识以为tmux用来分屏,后来才知道这只是tmux的一小功能,主要是用来保持会话的
在你想下班回家,然后有需要在开发机上执行一些耗时的任务,tmux和screen好处多多.
目前使用频率最高场景就是：

tmux开个会话，然后登陆跳板机，然后再detach掉，通过ControlMaster通过第一条会话来接管之后的ssh连接，这样今天一天就可以登陆线上机器不需要输入密码了。

### tmux

`tmux new -s fuck`     #创建会话

`tmux new -s fuck2 -d` #在后台建立会话

`tmux ls`              #列出会话

`tmux attach(a) -t fuck`  #进入某个会话

先按下ctrl + b

然后再按 ：

|命令|	作用|
|-|-|
|s|	显示已有tmux列表
|c|	创建一个新的窗口
|n|	切换到下一个窗口
|p|	切换到上一个窗口
|l|	最后一个窗口
|w|	显示打开的所有窗口，通过上下键可以选择
|数字|	直接跳到对应数字的窗口
|&	|退出当前窗口（大窗口）（还是ctrl + d来的实在）
|"	|竖屏新开一个窗口
|%	|横屏新开一个窗口哦u
|o	|在小窗口之间切换
|方向键|	小窗口之间移动
|! |关闭所有小窗口
|x	|关闭当前光标处的小窗口
|t	|时间
|,	|重命名窗口
|page up/page down	|可以滚动显示shell窗口 退出是 q

滚屏要进入copy-mode，即前缀+[，然后就可以用上下键来滚动屏幕，配置了vi快捷键模式，就 可以像操作vi一样来滚动屏幕，非常的方便。退出直接按‘q’键即可

### screen

### 参考

[1]<http://blog.chinaunix.net/uid-26285146-id-3252286.html>

[Tmux 入门介绍]<http://blog.jobbole.com/87278/>

[tmux]<http://mingxinglai.com/cn/2012/09/tmux/#top>
