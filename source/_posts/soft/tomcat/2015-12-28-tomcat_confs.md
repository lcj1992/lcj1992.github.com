---
layout: post
title: tomcat一些配置
categories: soft
tags: tomcat conf admin
---


### 添加管理员

conf/tomcat-users.xml

#### 6

    <role rolename="admin"/>
    <role rolename="manager"/>
    <user username="admin" password="admin" roles="admin,manager"/>

#### 7

    <role rolename="admin-gui"/>
    <role rolename="manager-gui"/>
    <user username="admin" password="admin" roles=" admin-gui , manager-gui "/>