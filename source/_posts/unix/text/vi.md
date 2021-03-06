---
layout: post
title: vi
date: 2015-12-27
categories: unix
tags:
    - vi  
    - vim
---

#### 编辑

|命令	|含义|
|-|-|
|r|	替换单一字符（在非insert模式下）
|J|	将当前行和下一行合并
|cc|	改变一整行
|cw	|改变最后一个单词
|c$	|改变光标到最后一个单词
|u	|undo
|ctrl-r|	redo
|.	|重复上一个命令
|~	|切换大小写
|g~iw|	切换当前单词的大小写
|gUiw|	当前单词大写
|guiw|	将当前单词改为小写
|\>\>	|向右缩进当前行
|\<\<	|向左缩进当前行
|==	|自动缩进当前行

#### 插入

|命令|含义|
|-|-|
|i|	在光标处进入插入
|I|	在当前行行首的非空字符处插入
|a|	在光标之后插入
|A|在行尾插入
|o|	在当前行下新开一空白行
|O|	在当前行前新开一空白行
|esc|	退出insert模式
|Cursor|  Movement
|ctrl-f	|向上翻页
|ctrl-b	|向下翻页
|%	|{} 、()、[]之间相互跳转，看代码利器
|w	|jump to end of words(标点)
|W|	jump by words
|e|	jump to end of words(标点)
|E|	jump to end of words
|b|	jump backward  by words(标点)
|B|	jump backward  by words
|0|	行首（含空白字符）
|^|	行首第一个非空字符
|$|	行尾
|gg|	第一行
|gd	|跳到方法或变量的声明处
|nG|	跳到第n行
|fx|	跳到当前行的x字符
|;|	重复上一个f命令
|tx|	和fx很像，但是跳到x之前的一个字符

#### 查找替换

|命令|　含义|
|-|-|
|/pattern|	模式匹配
|?pattern|	模式匹配(倒着来)
|n|	重复查找
|N|	重复查找倒序
|s/old/new/|替换当前航第一个old -> new
|s/old/new/g	|替换当前行所有
|s/old/new/gc	|每个替换都需要确认
|1,$s/old/new/g|替换第一行到最后一行, `1,$`等价于`%`

#### 剪切复制

|命令| 含义|
|-|-|
|dd|	剪切一行
|dw	|删除当前单词
|x|	删除光标当前字符
|X|	删除光标之前的字符
|D|	删除光标所在到行尾
|yy|	复制一行
|2yy|	复制两行
|yw	|复制光标当前单词
|y$|	复制光标到行尾所有字符
|p|	粘贴
|P|	在光标之前粘贴

#### 退出

|命令|含义|
|-|-|
|w	|保存
|wq|	保存并退出
|x|	保存并退出
|q|	退出，如果不保存会失败
|q!|	强制退出

#### 多文件操作

|命令| 含义|
|-|-|
|e xxx|	在一个新的buffer中编辑xxx
|bn|	切换到下一个buffer
|bd|	删除一个buffer（关闭文件）
|sp xxx|	在新的buffer中打开文件并分割窗口
|ctrl-w s	|分割窗口
|ctrl-w w	|切换窗口
|ctrl-w q	|退出窗口
|ctrl-w v	|垂直分割窗口
|table xxx|	编辑文件在一个新的tab
|gt|	下一个tab
|gT|	之前的tabs
|tabr|	第一个tab
|tabl|	最后一个tab
|tabm n	|move当前tab之后的第n个

#### 显示行号

*   方法一：

1、显示当前行行号，在VI的命令模式下输入:nu

2、显示所有行号，在VI的命令模式下输入:set nu

*   方法二：

使用vi编辑~/.vimrc文件，在该文件中加入一行"set nu"，添加内容不含引号， 命令如下：vi ~/.vimrc

*   方法三：

在UBUNTU中vi的配置文件存放在/etc/vim目录中，配置文件名为vimrc
在Fedora中vi的配置文件存放在/etc目录中，配置文件名为vimrc

在Red Hat Linux 中vi的配置文件存放在/etc目录中，配置文件名为vimrc

使用vi编辑该文件，在该文件中加入一行"set nu"，添加内容不含引号。如Ubuntu命令：vi /etc/vim/vimrc

#### 块编辑模式

1.ctrl + v 进入虚拟模式

2.上下移动

3.I （大写的i）进行编辑

4.ESC,使其对所有行生效

#### ps

显示当前文件名 `ctrl + g` 或者 `:f`

拷贝多行

方法一： 直接使用yy 和 p

`直接5yy 然后p`

方法二： 使用标签和缓冲区

本文件内：

    光标移到起始行，输入ma
    光标移到结束行，输入mb
    光标移到粘贴行，输入mc

    然后 :'a,'b co 'c
    把 co 改成 m 就成剪切了

不同文件内：

    光标移到起始行，输入ma
    光标移到结束行，输入mb

    然后:'a, 'b w filename
    在文件二：
    光标移到需要赋值的行，输入：
    :r filename

    光标移到结束行，输入mx
    光标移到起始行，输入"ay'x 复制放到a缓冲区中  
    光标移到需要复制的行，输入"ap

#### 参考

<http://www.freeos.com/guides/lsst/misc.htm#commonvi>

[vim命令速查卡一]<http://coolshell.cn/airticles/150.html>

[vim命令速查卡二]<http://coolshell.cn/airticles/5479.html>

[拷贝多行]<http://www.chinaunix.net/old_jh/4/910342.html>

[拷贝多行]<http://blog.csdn.net/xiyuan1999/article/details/5680102>
