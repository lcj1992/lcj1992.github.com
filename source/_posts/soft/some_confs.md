---
layout: post
title: 各种环境配置
date: 2014-11-18
categories: soft
tags:
    - soft
    - jdk
    - maven
    - git
---

### jdk
*	`JAVA_HOME` : &nbsp; &nbsp;		**D:\q\java\default**   mac: \`/usr/libexec/java_home\`
	(**你的jdk的安装目录，这里创建了一个软连接，指向了jdk的安装目录**)  
*	`path`:  &nbsp;&nbsp;			**;%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin**(**unix下是PATH**,之间已`:`分隔)
*	`CLASSPATH`: 	&nbsp;&nbsp;	**.;%JAVA_HOME%\lib;%JAVA_HOME%\lib\tools.jar**（**前面的.一定要加**）

### maven

*	`MAVEN_HOME`:	&nbsp;&nbsp;	**D:\bin\apache-maven**
*	`path`:		&nbsp;&nbsp;		**;%MAVEN_HOME%\bin**    
在.m2文件夹下注意要编辑settings.xml文件  
windows下是在C:\Users\xxx.li下
linux是在家目录下。
***mac 如果是brew 安装的软件,默认会在/usr/local/bin下, 在PATH中,所以不需要配置maven的环境变量***

### git
*	`GIT_HOME`:		&nbsp;&nbsp;	**D:\Program Files (x86)\Git**
*	`path`:		&nbsp;&nbsp;		**;%GIT_HOME%\cmd;**

### idea配置

file->settings

### node

```
brew install node
export NODE_PATH=/usr/local/lib/node_modules
```

### go

```
brew install go
export GOPATH=/usr/local/Cellar/go/1.6.2
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
```
### jekyll

```
***1.ubuntu14.04安装jekyll***

sudo apt-get install ruby
curl -sSL https://rvm.io/mpapis.asc | gpg2 --import -
curl -sSL https://get.rvm.io | bash -s stable
/usr/local/rvm/bin/rvm install 2.1


安装nodejs   yum install nodejs
sudo apt-get install nodejs

sudo gem install jekyll

sudo apt-get install ruby-redcarpet

sudo apt-get install ruby-kramdown

***2.mac**

brew install jekyll
```

### 使用

jekyll server -I
