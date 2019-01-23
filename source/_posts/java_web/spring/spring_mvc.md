---
layout: post
title: 从启动日志看spring mvc的流程
date: 2015-01-27
categories:  java_web
tags:
    - mvc
    - spring
---

### 原理图  

![spring mvc原理图](/images/web/springMvc.jpg)

### web项目的加载顺序

web项目的加载顺序：  
`context-param -> listener -> filter -> servlet`   
http://blog.csdn.net/jubincn/article/details/9115205      
结合着我们的项目分析一下spring mvc的启动流程：      

	<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:spring.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

### context-param 和 listener   

1.在tomcat启动之后，容器会创建一个ServletContext（上下文），应用范围内即整个项目都能使用，接着容器会将读到的<context-param>(我们项目中对应 classpath下的spring.xml)转化成键值对，并交给ServletContext。   

2.tomcat创建<listener></listener>中的类实例,即创建监听（备注：listener定义的类可以是自定义的类但必须需要继承ServletContextListener）。(我们的项目对应于org.springframework.web.context.ContextLoaderListener)。得到这个context-param的值之后,你就可以做一些操作了.日志中我们可以看到这这几句话：
	Pre-instantiating singletons in org.springframework.beans.factory.support.DefaultListableBeanFactory@12bc2e1a:defining beans [dataSource,jdbcTemplate,logService,transactionManager...    
DefaultListableBeanFactory帮我们做了Pre-instantiating singletons（这里会扫描spring.xml中的bean的注解（会扫描的注解有@resource，@Autowired,@Qualifier,、@PostConstruct,@PreDestroy等），按照其依赖关系，从小到大逐个实例化，而与在spring.xml配置文件中的顺序无关，可以通过注解@PostConstruct和@PreDestroy来进行验证)  
注意,这个时候你的WEB项目还没有完全启动完成.这个动作会比所有的Servlet都要早。    

3.这时候我们的context-param 和listener完成加载，项目部署成功.日志中可以看到Root WebApplicationContext: initialization completed in 599 ms，Artifact 2015training1:war exploded: Artifact is deployed successfully

### servlet  

DisPatcherServlet，表示以htm为后缀的都要经过这个分发器进行分发。
	<servlet>
        <servlet-name>web</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>web</servlet-name>
        <url-pattern>*.html</url-pattern>
    </servlet-mapping>   

自此请求已交给Spring Web MVC框架处理，因此我们需要配置Spring的配置文件,默认DispatcherServlet会加载WEB-INF/[DispatcherServlet的Servlet名字]-servlet.xml配置文件  

	<!-- 使用JSP作为模板资源 -->
    <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView" />
        <property name="prefix" value="/WEB-INF/jsp/" />
        <property name="suffix" value=".jsp" />
    </bean>

    <!-- 页面访问异常时，跳转到模板 -->
    <bean class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
        <property name="defaultErrorView" value="error/500"/>
        <property name="exceptionAttribute" value="ex"/>
    </bean>
    <context:component-scan base-package="com.spring.controller"/>

然后也会Pre-instantiating singletons(internalResourceViewResolver,SimpleMappingExceptionResolver,
requestMappingHandlerMapping,viewControllerHandlerMapping,requestMappingHandlerAdapter,  
simpleControllerHandlerAdapter,handlerExceptionResolver,jsonBodyExceptionResolver，SimpleMappingExceptionResolver等等)  
再然后requestMappingHandlerMapping这个实例会去controller包中扫描其中的类：并扫描注解，做映射：  
![对应日志](/images/web/dispatch.jpg)  
requestMappingHandlerAdapter将HandlerExecutionChain中的处理器（DemoController）适配为requestMappingHandlerAdapter FrameworkServlet 'web': initialization completed    
当传入http://localhost:8080/user/jsp/user.htm?name=bob，  
根据映射关系，执行DemoController中的get方法，返回一个ModelAndView("user/info").addObject("user", user);然后根据ViewResolver中的配置  前缀[逻辑视图名]后缀 也即/WEB-INF/jsp/user/jsp/info.jsp 并传入对象user，渲染，展示，ok了。
