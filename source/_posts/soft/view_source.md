---
layout: post
title: 我是这样看源码的
date: 2016-02-28
categories: soft
tags: vim ctags cscope doxygen graphviz code2ebook
---





*   [doxygen+graphviz](#doxygen_graphviz)


### vi分屏 

***1.直接用终端打开多个文件***

a.横向分屏

    vi -o file1 file2

b.纵向分屏

    vi -O file1 file2

***2.如果已经用vi打开了一个文件,想要在窗口中同时打开另一个文件***

a.横向分屏 vs或vsplit

    :vs filename

b.纵向分屏 sp 或split

    :sp filename

如果文件名不存在,会新建文件并打开

***3.关闭窗口***

a.关闭光标所在窗口

    :q 或 :close

b.关闭除光标所在的窗口之外的其他窗口

    :only

c.关闭所有窗口

    :qa

***4.切换窗口***

ctrl + w w 或ctrl + w + 方向键

### ctags

***1.安装***

ubuntu: `sudo apt-get install ctags`
mac: 自带的ctags 不支持-R递归,所以想要-R的需要自己安装

1.  http://ctags.sourceforge.net/  下载源码
2.  tar zxvf ctags-5.8.tar.gz
3.  sudo ./configure && make all && sudo make install

***2.使用***

进入源码根目录,创建tags `ctags -R *`

|快捷键|作用|
|---|---|
|ctrl ] |跳到光标下方的tag|
|ctrl t |调回tag栈|
|:ts <tag> <RET> |跳到指定的tag|
|:tn| 跳到下一个tag的定义|
|:tp| 跳到上一个tag的定义|
|:ts| 列举上一个tag所有的定义|

### cscope

***1.安装***

ubuntu `sudo atp-get install cscope`

mac `brew install cscope`

***2.使用***

首先需要为你的代码生成一个cscope数据库。生成数据库很简单，在你的项目根目录运行下面的命令：

     cscope -Rbq

这个命令会生成三个文件：cscope.out, cscope.in.out, cscope.po.out。

其中cscope.out是基本的符号索引，后两个文件是使用”-q"选项生成的，可以加快cscope的索引速度,-R: 在生成索引文件时，搜索子目录树中的代码

|命令|作用|
|--|--|
|b| 只生成索引文件，不进入cscope的界面|
|q|生成cscope.in.out和cscope.po.out文件，加快cscope的索引速度
|k| 在生成索引文件时，不搜索/usr/include目录
|i| 如果保存文件列表的文件名不是cscope.files时，需要加此选项告诉cscope到哪儿去找源文件列表。可以使用“-”，表示由标准输入获得文件列表。
|I dir| 在-I选项指出的目录中查找头文件
|u| 扫描所有文件，重新生成交叉索引文件


***3.在vim中使用cscope***

1 、用vim编辑的时候 ：

    vim filename.c

2 把生成的cscope文件导入到vim中来

    :cs add /路径/cscope.out 代码所在目录

我习惯现切换到代码所在目录再操作，所以直接使用：

    :cs add cscope.out 就可以了。

避免每次都加在cscope，可以在`.vimrc`中添加cscope的配置，自动添加

    if has("cscope")
      set csprg=/usr/local/bin/cscope
      set csto=0
      set cst
      set csverb
      if filereadable("cscope.out")
        cs add cscope.out
      elseif $CSCOPE_DB !=""
        cs add $CSCOPE_DB
      endif
      set csverb
    endif

3 查看是否已经连接到对应数据库

    :cs s

4 `cs f s xxxx` 查找xxxx出现的地方，它能详细列出哪些文件的哪行的哪个函数引用，以及该行的内容.

s: 查找C语言符号，即查找函数名、宏、枚举值等出现的地方

    :cs f s main

g: 查找函数、宏、枚举等定义的位置。

    :cs f g main

d: 查找本函数调用的函数

    :cs f d main

c: 查找调用本函数的函数

    :cs f c main 会输出找到没有匹配的结果，因为没有函数调用main函数

t: 查找指定的字符串

    :cs f t string

f: 查找并打开文件

    :cs f f filename 将会打开文件名为FIANME的文件

### gdb

安装gdb

    brew install gdb

如果mac上gdb提示签名错误，参照[这篇](http://blog.csdn.net/powerlly/article/details/30323015)解决下。
已redis为例

    ps -ef | grep redis
    gdb -p redisPid

 具体可参考dash（有了dash和alfred这对组合，简直太方便了，强烈推荐）

### doxygen_graphviz

doxygen 可以为我们生成代码类图，调用关系等

1. Wizard:
   * Project:选择源码目录(记着勾选 递归scan recursively)，生成的html存放的目录
   * Mode：All Entities，Include cross-referenced source code in the output,对应的代码类型
   * Output: with navigation panel
   * Diagrams: use dot tool from the graphviz package  ->call graphs,called by graphs
2. Expert:
   * Build: extract_all extract_private extract_package extract_static extract_local_classes extract_local_methods
   * Dot: class_diagrams DOT_PATH:/usr/local/bin
3. run doxygen

graphviz,下载[graphviz](http://www.graphviz.org/Download..php)
然后就可以用doxygen生成源代码文档，并通过grapphviz生成调用关系图，类图等

### code2ebook

code2ebook是偶像写的一个源代码到电子书的转换工具，也很赞[github](https://github.com/agentzh/code2ebook)

### 参考
