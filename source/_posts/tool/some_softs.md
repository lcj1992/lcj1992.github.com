---
layout: post
title: 一些有趣的工具
date: 2016-03-21
categories: tool
tags:
    - alfred
    - dash
    - oh-my-zsh
    - zsh
    - z
    - switchHosts
    - theFuck
    - z
    - homebrew
    - iterm2
    - reeder
    - xmind
    - omnigraffle
---

### alfred

*   快捷键 opt + space
*   打开应用程序
*   查找文件 回车定位文件 cmd + 回车 定位文件所在文件夹
*   **通过find、open、in等关键词搜索文件,find是定位文件，open是定位并打开文件，in是在文件中进行全文检索**
*   计算器
*   输入>即可直接运行shell命令 (收费)

#### 我用的workflow

![alfred_workflow.png](/images/tool/alfred_workflow.png)

### switchHosts

windows和mac都有,测试切换hosts，之前是手工改/etc/hosts的，后来感觉还是慢。[github地址](https://github.com/oldj/SwitchHosts)

### Dash

AppStore中就有,各种文档,每个开发必备, [破解版](http://www.sdifen.com/dash402.html)

### homebrew

mac homebrew 安装

`/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`

homebrew有个插件也不错cask,主要用来管理非终端软件

安装cask

    brew install caskroom/cask/brew-cask

使用cask来安装谷歌浏览器

    brew cask install google-chrome

### iterm2

1.  设置热键,在iterm中设置F12,`preference-key-hotkey-show...`
2.  `cmd + enter` 全屏显示
3.  `cmd + 左/右,cmd + 数字` 切换标签
4.  `view-show tabs in fullscreen`,在全局模式下显示标签
5.  切分屏幕：`cmd + d` 水平切分，`cmd + Shift + d` 垂直切分
6.  `cmd + ;`弹出自动补齐窗口
7.  `cmd + Shift + h`弹出历史记录窗口
8.  `cmd + option + e`所有tab查找
9.  `cmd + /`高亮当前鼠标的位置
10. `preferences->profiles->keys` 最下面的Left/Right Option，把normal改成esc Option + B/F 光标逐词移动
11. `cmd + l` 编辑当前session，结合着cmd + o 支持将tab命名，并快速切换。

1,2使用场景: 新开一个桌面,专门放终端,然后cmd+enter全屏显示,然后切换到终端F12,对于经常要登服务器的码农们,简直不要太赞哦

### idea

1.   mybatis plugin [破解](http://download.csdn.net/download/jianglie/9562959)
2.   maven-helper
3.   jvm debugger memory view
4.   ini
5.   gitignore
6.   markdown
7.   lombok

### reeder

mac上我的第一个掏钱的软件,买亏了。

### snip

snip 腾讯的,mac自带的不太好用啊

### zsh-终极shell

***1.安装zsh***

*   `cat /etc/shells`你会发现mac是有自带zsh的,但是和linux一样，默认的终端都是bash(ubuntu安装`sudo apt-get install zsh`)
*   安装完成之后设置当前用户使用zsh: `chsh -s /bin/zsh`。但因为其配置复杂，所以很多人无人问津，
*   然后有人开发了个让你快速上手zsh的项目叫做［oh－my－zsh］

***2.安装oh－my－zsh**://github.com/robbyrussell/oh-my-zsh)

安装：`sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"`

1.	quick-look filename 可以直接预览文件，man-preview grep 可以生成 grep手册 的pdf 版本等
2.	强大的历史纪录功能，输入 grep 然后用上下箭头可以翻阅你执行的所有 grep 命令
3.	各种补全：路径补全、命令补全，命令参数补全，插件内容补全等等。触发补全只需要按一下或两下 tab 键，补全项可以使用 ctrl+n/p/f/b上下左右切换。比如你想杀掉 java 的进程，只需要输入 kill java + tab键，如果只有一个 java 进程，zsh 会自动替换为进程的 pid，如果有多个则会出现选择项供你选择。ssh + 空格 + 两个tab键，zsh会列出所有访问过的主机和用户名进行补全

***3.使用zsh***

1.	安装z插件，只需在～／.zshrc的plugin中添加`plugins=(git xx z)`,然后你就可以z xx目录之间随意跳转了.如果你没有使用oh-my-zsh框架，也可以使用z命令，下载z.sh,[git地址](https://github.com/rupa/z/),然后在环境变量中指定z.sh的路径`. /home/lcj/tool/z.sh`

### theFuck

纠正命令
[github地址](https://github.com/nvbn/thefuck)

### z

目录跳转,真的是神器

[github地址](https://github.com/rupa/z)

### omnigraffle

mac上的`visio`

### xmind

梳理思路，总结知识点，脑图。

### edrawMax

[比omigraffle好用](http://www.sdifenzhou.com/edrawmax84.html)

### go2Shell

切换终端为iterm，命令行输入一下：

open -a Go2Shell --args config

## 参考

[终极 Shell]<http://macshuo.com/?p=676>

[神兵利器——Alfred]<http://www.cnblogs.com/chijianqiang/p/alfred.html>

[高效程序员的MacBook工作环境配置]<http://www.codeceo.com/article/programmer-macbook-workplace.html>

[我在用的mac软件(1)--终端环境之iTerm2]<http://foocoder.com/blog/wo-zai-yong-de-macruan-jian.html/>

[你应该知道的iterm2使用方法]<http://wulfric.me/2015/08/iterm2/>

[cask官网]<https://caskroom.github.io/>

[brew官网]<http://brew.sh/>

[iterm2有什么酷功能?]<http://www.zhihu.com/question/27447370>

[mac开发配置手册]<https://aaaaaashu.gitbooks.io/mac-dev-setup/content/>

[5 Unexpectedly Useful Command Line ]<http://zeroturnaround.com/rebellabs/5-unexpectedly-useful-command-line-tools-you-might-overlook/>

[GitHub 上有什么好的或者有趣的 Shell 项目？]<http://www.zhihu.com/question/28182203>
