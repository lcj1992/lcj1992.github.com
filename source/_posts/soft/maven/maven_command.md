---
layout: post
title: maven基本命令
date: 2015-12-28
categories: soft
tags: maven
---

#### 基础
SNAPSHOT版本会实时比对仓库版本。即使本地有了同一版本jar包，还是会比对仓库。所以SNAPSHOT比较试用在开发阶段，需频繁更改代码，而不用更改版本

#### 常用命令

`mvn clean package -Dmaven.test.skip=true` 打包之前先clean，不执行test

`mvn clean package -Pdev`指定资源

`mvn deploy`部署到maven仓库（部署子模块也得从工程根路径下进行）

`mvn dependency:tree -Dverbose -Dincludes(-Dexcludes)=groupId:artifactId` 检查依赖

`mvn enforcer:enforcer` 规则校验

`mvn test -Dmaven.test.skip=false` 测试
