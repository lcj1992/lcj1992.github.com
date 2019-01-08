---
layout: post
title: dubbo基本配置
categories: soft
---

#### 流程
以proxy invoke crawler的服务为例。

provider

    <!-- providers-->
    <dubbo:application name="autoticket-common-provider" owner="xiang.zhu" organization="QDev"/>
    <dubbo:registry id="commonServiceProvider" address="zk.xxx.com" protocol="zookeeper" group="autoticket_common_prod"/>

    <bean id="taskProcessService" class="com.xxx.common.task.service.impl.TaskProcessService"/>
    <dubbo:service id="taskProcessService_dubbo" interface="com.xxx.common.task.service.ITaskProcessService"
                   ref="taskProcessService" version="1.0.0" group="autoticket" registry="commonServiceProvider"/>

consumer

    <!-- Proxy和Crawler之间消息服务 -->
    <dubbo:registry address="zk.xxx.com" protocol="zookeeper" group="autoticket_common_prod" id="messageZK" default="false"/>
    <dubbo:reference id="messageService" interface="com.xxx.common.task.service.ITaskProcessService" registry="messageZK" version="1.0.0" group="autoticket"/>
   

proxy 调用 commonService 的 ITaskProcessService 这个服务。

#### 参数
红色为必须参数

`application`

name 当前应用名称，用于注册中心计算应用间依赖关系

owner

organization

provider（这里是属性）

threads  线程池数量

timeout  超时时间

cluster？

retrys     重试次数

loadbalance 均衡策略

executes

`registry`

address 注册中心服务器地址，如果地址没有端口缺省为9090，同一集群内的多个地址用都好分隔，如：ip:port,ip:port,不同汲取你的注册中心，请配置多个<dubbo:registry/>

id

group      注册中心的group

protocol

protocol  name协议名称

`service`

id

interface 服务接口名

ref  服务对象实现引用

version

group 服务分组（一个接口可以有多个实现，通过group来区分）

registry 注册中心id (缺省向所有registry注册)

`reference`
id  服务引用beanId

interface 服务接口名

version

registry (缺省将从所有registry获服务列表后合并结果)

#### 参考

[dubbo官网]<http://dubbo.io>