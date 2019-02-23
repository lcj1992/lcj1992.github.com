---
layout: post
title: sublime插件
date: 2016-03-11
categories: tool
tags:
    - sublime
---

首先需要安装包管理插件,方便安装其他插件

### package control

    1.  使用快捷键 ctrl + \` 或者 view -\> show console ,执行文末链接中python代码
    2.  下载Package Control.sublime-package,并将其拷贝到/Users/fool/Library/Application Support/Sublime Text 3/Packages下
    3.  重启

### markdown
*   安装markdown插件
    1.  cmd + shift + p 输入install package ,然后回车
    2.  输入 markdown pre 然后回车
*   安装markdown高亮插件
    1.  跟上边安装markdown插件一样, Monokai extended
*   使用    
    cmd + shift + p ,然后输入 mdp ,选择你要的操作(我一般是在浏览器中看), 然后选择你要的风格 github的,还是原生的(我是markdown)
    cmd + b 实时编译,然后刷新浏览器

### sublimerge

一个很棒的比对工具，支持git哟

*	ctrl + alt + D 调出其选项，

### json插件

sublime text json 格式化插件安装

1.  进入插件目录`cd Library/Application\ Support/Sublime\ Text\ 3/Packages/`
2.  git clone `git clone https://github.com/dzhibas/SublimePrettyJson.git`
3.  重启

一些好的插件 : <https://github.com/mytharcher/SimpleGray/blob/master/README.md>

### 快捷键

#### 查找命令

文件之间的切换，文件间的查找，跳到某个文件第某行

`cmd + p`

Type part of a file name to open it.

1.  @ to jump to symbols,  `tp@rf`  会匹配test.py中的read_file
2.  \# to search within the file,  `tp#order` 会匹配test.py中的orderId
3.  : to go to a line number.  `tp:100` 会跳到test.py的第100行

#### 调出命令面板

`cmd + shift + p`

1.  ss 设置文件的语法格式

#### 多窗口编辑

view－layout ，多窗口编辑

opt ＋ cmd ＋1

File－New View into File 多视图

#### 多选

shift ＋ cmd ＋ L  选择
然后cmd ＋ d 选择下一处
然后进行重命名

ctrl + cmd + j 格式化json
ctrl + F5   排序

注册码：见参考[2]

## 参考

[1]<https://packagecontrol.io/installation>

[2]<http://9iphp.com/web/html/sublime-text-3-license-key.html>

[sublimerge官网]<http://www.sublimerge.com/>