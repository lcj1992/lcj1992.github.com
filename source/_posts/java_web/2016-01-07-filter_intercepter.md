---
layout: post
title: 过滤器和拦截器
categories: java_web
tags: filter intercepter
---

#### 概述

1.  interceptor是spring的,所以interceptor只对走spring的请求有效
2.  filter是tomcat自己的

#### interceptor

    <mvc:interceptors>
       <!-- 所有走spring的请求都不缓存 -->
       <bean id="webContentInterceptor" class="org.springframework.web.servlet.mvc.WebContentInterceptor">
          <property name="cacheSeconds" value="0"/>
          <property name="useExpiresHeader" value="true"/>
          <property name="useCacheControlHeader" value="true"/>
          <property name="useCacheControlNoStore" value="true"/>
       </bean>
       <!-- 上下文拦截器（保存site等信息到线程变量中） 强制登陆等 -->
       <bean class="com.xxx.system.ContextInterceptor">
          <property name="forceLoginExcludePattern">
             <array>
                   <value>/qmonitor.jsp</value>
                   <value>/favicon.ico</value>
             </array>
          </property>
       </bean>
    </mvc:interceptors>

#### filter

    <filter>
        <filter-name>checkAccess</filter-name>
        <filter-class>com.xx.filter.UserFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>checkAccess</filter-name>
        <url-pattern>/xx.jsp</url-pattern>
    </filter-mapping>
