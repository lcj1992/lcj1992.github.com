---
layout: post
title: git基础命令
date: 2014-11-17
categories: soft
tags:
    - git
---

#### git原理简图：

![git原理简图](/images/git/git.png)

#### ssh设置

*	配置用户名     
	`git config --global user.name "lcj1992" `
*	配置邮箱     
	`git config --global user.email "1227627228@qq.com"`  
*	生成秘钥    
	`ssh-keygen -t rsa -C "1227627228@qq.com"`
*	按三个回车，密码为空（不需要输入密码，你要输入也不强求）  
*	然后把生成的id_rsa.pub内容粘贴到github个人中心的账户设置的ssh key里面，最好以机器名+git服务器作为名字加以区分

#### push受限制
换做ssh方式aaa

    git remote set-url --add origin git@xxxxx.git
    git remote set-url --delete origin https://xxx.git
    ssh-add ~/.ssh/id_rsa

按照上边的ssh设置之后，这样更换之后终端可以使用了，但是idea的ctrl + T(fetch)  ctrl + shift + K(push) 并不管用，进行如下设置

    settings-->Version Control-->Git ,and then, In the SSH executable dropdown, choose Native

#### push到远程别的分支

    git push origin local_branch_a:remote_branch_b

#### 删除远程指定分支

    git push origin :master
    git push origin --delete master //两者等价

#### 远程分支覆盖本地分支

    git fetch --all
    git reset --hard origin/brancha

#### 强制回到某一次提交

    git reset --hard shaiID
    git push --force 强制覆盖远端分支

#### git reset rebase revert

    reset 重置
    revert 还原
    rebase　变基

#### git本地工程关联远端仓库

    git init
    git remote add origin xxxx.git

#### git diff

    git diff master 所在分支diff本地master分支
    git diff origin/master 所在分支diff远端master分支

#### git常规使用

*	在github/gitlab上新建一个reposrity      
*	从远程仓库git@github.com:lcj1992/lcj1992.github.com.git克隆一份到本地Blogs，这里边第一个lcj1992是我的用户名  
	`git clone git@github.com:lcj1992/lcj1992.github.com.git Blogs` （可以添加 -o 参数，来代替长长的仓库地址，默认是origin）   
	`cd Blogs`
*	新建一个分支  
	`git branch prd_1`
*	切换到自己的分支  
	`git checkout prd_1`
*   新建分支并切换到该分支
    `git checkout -b prd_1`
*	将自己本地的分支prd_1 push到远程服务器origin上  
	`git push origin prd_1` （origin是默认，在克隆时候-o参数可以任意制定）
*	添加文件到本地仓库  
	 `git add filename`(文件名，如果带的是参数 * 表明这个文件夹下的全部add）
*	提交到本地仓库  
    `git commit -m "标签说明"`（如果没有添加文件，可以这里的add 和commit合并了，在commit时增加 -a 参数）
*	推送到远程服务器  
    `git push`（默认推送到与当前分支相关联的远程分支上）
*	切换https到ssh  
	`git remote set-url origin xxxx`
*   比较本地和远程某分支差异  
    `git diff origin/branchxxx`
*   删除本地所有未提交的更改  
    `git checkout . && git clean -xdf`

#### 发布回滚后重新merge　master导致代码丢失

1.  merge master分支代码（就是这一步会冲掉本次新改动的代码）
2.  提交或push merge 后的代码，理论上提交了就成．
3.  git revert [commit] 或git revert HEAD~[NUM]：commit为回滚时候产生的commit版本,就是带着qa名字的那个,HEAD~[NUM]为回滚到之前第几个版本
4.  解决冲突
5.  该add add 该commit commit然后push就ok了

#### 参考

[如何删除本地所有未提交的更改](https://www.v2ex.com/t/66718)
