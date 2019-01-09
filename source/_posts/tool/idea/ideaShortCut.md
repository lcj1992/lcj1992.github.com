---
layout: post
title: idea快捷键
date: 2015-11-25
categories: tool
tags: idea
---



    
    
    *   [usage　search](#usage_search)
    
    
    
    
    *   [vcs/local history](#vcs)
    
    

## 代码颜色 

![git](/images/git/git.png)

|颜色 |状态|
|--|--|
|蓝色	|工作区和本地仓库不一致|
|绿色	|已经add到index中，但是不在本地仓库中|
|白色	|工作区和本地仓库代码一致|
|红色	|只在本地工作区|

## 快捷键 

<h3 id="in practice">实用至上</h3>

Command,Shift,Option,Ctrl参看[mac按键](/2016/02/03/mackeyboard)

|mac|pc|作用|
|---|---|---|
|Cmd + E |Ctrl + E	|最近使用的文件(不一定编辑)|
|Cmd + Shift + O|Ctrl + Shift + N	|跳转到指定文件或者目录|
|Cmd + Option + O|Ctrl + Shift + Alt + N|	跳转到指定的符号|
|Cmd + 1|Alt + 1	|project工具窗口|
|Esc|Esc	|返回编辑器|
|Cmd + B |Ctrl + U	|跳转到父类或者接口|
|Cmd + Option + B |Ctrl + Alt + B|	跳转到实现|
|Ctrl + H |Ctrl  + H|类的继承关系|
|Ctrl + Option + H |Ctrl + Shift  + H　我的给改成Ctrl + Alt + Shift + H了|方法的继承关系|
|Cmd + Shift + H |Ctrl + Alt + H	|方法的调用关系|
|Cmd + Shift + Option + U |Ctrl + Alt  + U	|UML图展示类的层次关系 （F5 美化 空格 添加新类）|
|Cmd + Option + F7 |Alt + F7|查看使用处 show usages|
|Cmd + Shift + E |Ctrl + Shift + E	|最近被编辑过的文件|
|Cmd + Shift + \`|Ctrl + Alt + \[|上一个工程|
|Cmd + \` |Ctrl + Alt + \]|下一个工程|
|Cmd + shift + F12|最大化当前窗口|


<h3 id="edit">Editing</h3>

|mac| pc | 作用  |
|---|---| ---|
|Ctrl + Space |Ctrl + Space| Basic code completion (the name of any class,method or variable)|
|Ctrl + Shift  + Space |Ctrl + Shift + Space| Smart code completion (filters the list of methods and variables by expected type)|
|Cmd + Shift + Enter |Ctrl + Shift + Enter| Complete statement
|Cmd + P |Ctrl + P |Parameter info (within method call arguments)
|Ctrl + J |Ctrl + Q |快速文档查看
|Shift + F1 |Shift + F1 |External Doc
|Cmd + mouse over code |Ctrl + mouse over code |Brief Info
|Cmd + F1 |Ctrl + F1 |Show descriptions of error or warning at caret
|Cmd + N,Ctrl + N |Alt + Insert |产生代码(get/set方法，构造器，hasCode/equals，toString以及继承来的方法)
|Ctrl + O |Ctrl + O |重写方法
|Ctrl + I |Ctrl + I |实现方法
|Cmd + Alt + T |Ctrl + Alt + T |Surround with… (if..else, try..catch, for,synchronized, etc.)
|Cmd + / |Ctrl + / |行注释
|Cmd + Shift + / |Ctrl + Shift + / |块注释
|Opt + 上 |Ctrl + W |递增选择代码
|Opt + 下 |Ctrl + Shift + W |递减当前选择
|Ctrl + Shift + Q |Alt + Q |Context info
|Opt + Enter |Alt + Enter |Show intention actions and quick-fixes
|Cmd + Opt + L |Ctrl + Alt + L |格式化代码
|Ctrl + Opt + O |Ctrl + Alt + O |整理导入
|Ctrl + Opt + I |Ctrl + Alt + I |Auto-indent line(s)
|Tab ／ Shift ＋ Tab|Tab / Shift + Tab| Indent/unindent selected lines
|Cmd + X  |Ctrl + X or Shift + Delete |剪切当前行或者所选区域
|Cmd + V |Ctrl + V or Shift + Insert |复制
|Cmd + Shift + V |Ctrl + Shift + V |Paste from recent buffers...
|Cmd + D |Ctrl + D |复制当前行或者所选区域
|Cmd + Del |Ctrl + Y |删除当前行
|Ctrl + Shift + J |Ctrl + Shift + J |合并两行
|Cmd + Enter |Ctrl + Enter |增加空行
|Shift + Enter |Shift + Enter |增加新行
|Cmd + Shift + U|Ctrl + Shift + U | 切换大小写
|Cmd + Shift + ]/[|选择代码直到代码块开始处和结尾处|
|opt + Del|Ctrl + Delete |删除以当前光标为首的单词
|opt + Fn + DEl|Ctrl + Backspace |删除以当前光标为尾的单词
|Cmd + NumPad+/-  |Ctrl + NumPad+/- |合并／扩展代码块
|Cmd + Shift + NumPad+ |Ctrl + Shift + NumPad+ |Expand all
|Cmd + Shift + Numpad- |Ctrl + Shift + NumPad- |Collapse all
|Cmd + W |Ctrl + F4 |Close active editor tab|

<h3 id="search_replace">Search/Replace</h3>

|mac|pc| 作用  |
|----| -----  | ----- |
|Double Shift|Double Shift | 全局搜索
|Cmd + F|Ctrl + F |文件内查找
|Cmd + G|F3 |查找下一个
|Cmd + Shift + G|Shift + F3 |查找上一个
|Cmd + R|Ctrl + R |文件内替换
|Cmd + Shift + F|Ctrl + Shift + F |全局查找
|Cmd + Shift + R|Ctrl + Shift + R |全局替换
|Cmd + Shift + S|Ctrl + Shift + S |Search structurally (Ultimate Edition only)
|Cmd + Shift + M|Ctrl + Shift + M |Replace structurally (Ultimate Edition only)

<h3 id="usage_search">usage search</h3>

|mac| pc | 作用  |
|---| -----  | ----- |
|Opt + F7|Alt + F7 / Ctrl + F7 |Find usages / Find usages in file
|Cmd + Shift + F7|Ctrl + Shift + F7 |Highlight usages in file
|Cmd + Opt + F7|Ctrl + Alt + F7 |Show usages

### 编译运行 

|mac| pc| 作用  |
|---| -----  | ----- |
|Ctrl + Opt + R|Alt + Shift + F10 |Select configuration and run
|Ctrl + Opt + D|Alt + Shift + F9 |Select configuration and debug
|Ctrl + R|Shift + F10 |运行
|Ctrl + D|Shift + F9 |调试


### 调试 

|mac| 快捷键| 作用  |
|---| -----  | ----- |
|F8|F8 |单步跳过
|F7|F7 |单步跳入
|Shift + F7|Shift + F7 |Smart step into
|Shift + F8|Shift + F8 |跳
|Opt + F9 |Alt + F9 |运行到光标
|Opt + F8|Alt + F8 |Evaluate expression
|Cmd + Opt + R|F9 |执行到断点
|Cmd + F8|Ctrl + F8 |切换断点
|Cmd + Shift + F8|Ctrl + Shift + F8 |查看所有断点,当鼠标该行有断点时，是更改该行的条件

### 导航 

|mac| pc| 作用  |
|----|-----  | ----- |
|Cmd + O|Ctrl + N |跳到给定名字的类
|Cmd + Shift + O|Ctrl + Shift + N |跳到给定名字的文件
|Cmd + Opt +  O |Ctrl + Alt + Shift + N |跳到给定名字的符号(注意，以上这三个快捷键都可以使用缩写，比如要找一个名字为QunarSystem的类，可以只输入QS)
|Ctrl + 左／右|Alt + Right/Left |Go to next/previous editor tab
|..|F12 |Go back to previous tool window
|Esc|Esc |Go to editor (from tool window)
|Shift + Esc|Shift + Esc |Hide active or last active window
|Cmd + Shift + F4|Ctrl + Shift + F4 |Close active run/messages/find/... tab
|Cmd + L|Ctrl + G |跳到第几行
|Cmd + E|Ctrl + E |最近用到的文件
|Cmd + Opt + 左／右|Ctrl + Alt + Left/Right |前进/后退
|Cmd + Shift + Del|Ctrl + Shift + Backspace |Navigate to last edit location
|Opt + F1|Alt + F1 |Select current file or symbol in any view
|Cmd + B|Ctrl + B or Ctrl + Click |跳到声明处
|Cmd + Opt + B|Ctrl + Alt + B |跳到实现处
|Cmd + Y|Ctrl + Shift + I |Open quick definition lookup，快速查看定义
|Ctrl + Shift + B|Ctrl + Shift + B |跳到类型声明处
|Cmd + U|Ctrl + U |跳到超类或超方法
|Ctrl + 上／下|Alt + Up/Down |跳到上一个方法/下一个方法
|Cmd + ]/[|Ctrl + ] / [ |光标移动到代码块开始/结束
|Ctrl + H|Ctrl + H |类型层次
|Cmd + Shift + H|Ctrl + Shift + H |方法层次
|Ctrl + Opt +H|Ctrl + Alt + H  (我给改成Ctrl + Alt + Shift + H)了|调用层次
|F2 /Shift + F2|F2 / Shift + F2 |Next/previous 错误
|F4|F4 | 查看源码
|F3|F11 |切换书签
|Opt + F3|Ctrl + F11 |Toggle bookmark with mnemonic
|Ctrl + #[0-9]|Ctrl + #[0-9] |跳到第几个标签
|Cmd + F3|Shift + F11 |显示标签
||Alt + F7 |查看方法使用

### 重构 

|mac| pc| 作用  |
|-----| -----  | ----- |
|F5|F5 |Copy
|F6|F6 |移动
|cmd + Del|Alt + Delete |Safe Delete  
|Shift + F6|Shift + F6 |重命名
|Cmd + F6 |Ctrl + F6 |Change Signature
|Cmd + Opt + N|Ctrl + Alt + N |Inline
|Cmd + Opt + M|Ctrl + Alt + M |提取方法
|Cmd + Opt + V|Ctrl + Alt + V |提取变量
|Cmd + Opt + F|Ctrl + Alt + F |提取字段
|Cmd + Opt + C|Ctrl + Alt + C |提取常量
|Cmd + Opt + P|Ctrl + Alt + P |提取参数

### vcs/local history 

|mac| pc| 作用  |
|---| -----  | ----- |
|Cmd + K|Ctrl + K |Commit project to VCS
|Cmd + T|Ctrl + T |Update project from VCS
|Shift + Opt + C|Alt + Shift + C |View recent changes

### 模板 

|mac| pc| 作用  |
|---| -----  | ----- |     
|Cmd + Opt + J|Ctrl + Alt + J |Surround with Live Template
|Cmd + J |Ctrl + J |插入现有的模板(geti,psf,psfi,psfs,psvm,St等)
|iter|iter | jdk1.5之后迭代风格 for-each
|inst|inst | 用instancof检测类的类型，并向下转
|itco|itco | 迭代java.util.Collection的元素
|itit|itit | 迭代java.util.Iterator的元素
|itli|itli | 迭java.util.List的元素
|psf|psf | public static final
|thr|thr | throw new

### 通用 

|mac| pc| 作用  |
|---| -----  | ----- |
|Cmd + Opt + Y|Ctrl + Alt + Y |Synchronize
|Cmd + Shift + F12|Ctrl + Shift + F12 |切换最大化编辑模式
|Cmd + Ctrl + F12||全屏|
|Opt + Shift + F|Alt + Shift + F |Add to Favorites
|Opt + Shift + I|Alt + Shift + I |以当前配置检查代码
|Ctrl + \`|Ctrl + \` |Quick switch current scheme
|Cmd + ,|Ctrl + Alt + S |打开设置对话框
|Cmd + ;|Ctrl + Alt + Shift + S |打开系统结构对话框
|Cmd + Shift + A|Ctrl + Shift + A |Find Action
||Alt + Shift + insert | 块编辑模式
|Cmd + Opt + U|Ctrl + Alt + U | 生成类图,悬浮模式
|Cmd + Opt + Shift + U|Ctrl + Alt + Shift + U | 生成类图,新开标签页


IDEA已经为你准备了很好的快捷键文档：菜单栏 -> Help -> Default Keymap Reference   
idea生成builder代码    
在Editor里右键----Refactor---Replace Constructor With Builder
