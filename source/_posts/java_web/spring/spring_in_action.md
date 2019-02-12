---
layout: post
title: spring实战
date: 2016-01-07
categories: java_web
tags:
    - spring
    - cglib
    - dynamicProxy
---

本文就是用，不讲任何源码，简易版手册

#### 读取配置文件

    1.<bean class="org.springframework.context.support.PropertySourcesPlaceholderConfigurer">
        <property name="locations">
            <list>
                <value>classpath*:*.properties</value>
                <value>classpath*:*/*.properties</value>
            </list>
        </property>
    </bean>
    ----------
    2.<context:property-placeholder location="classpath*:*.properties" file-encoding="UTF-8"/>

#### 使用aspectj aop来初始化上下文

    <!-- 使用aop来初始化上下文 -->
    <aop:aspectj-autoproxy proxy-target-class="true"/>
        <bean id="AopForContext" class="com.xxx.util.AopForContext">
    </bean>

#### 开启spring mvc controller

1.  如果没有@ResponseBody的话，spring mvc的返回走的是spring的视图渲染器，可以设置model，可以forword://xxxUrl?orderNo
2.  如果有ResponseBody的话，返回return的结果，可以设置message-Converter处理返回值例如spring自带的MappingJackson2HttpMessageConverter


        <!-- Enables the Spring MVC @Controller programming model -->
        <mvc:annotation-driven>
            <mvc:message-converters>	<!--支持的media类型s-->
                <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                    <property name="supportedMediaTypes">
                        <list>
                            <value>text/plain;charset=UTF-8</value>
                            <value>text/xml;charset=UTF-8</value>
                            <value>text/html;charset=UTF-8</value>
                        </list>
                    </property>
                </bean>	<!--json格式的序列化与反序列化-->
                <bean id="mappingJacksonHttpMessageConverter"
                      class="org.springframework.http.converter.json.MappingJacksonHttpMessageConverter">
                    <property name="supportedMediaTypes">
                        <list>
                            <value>application/json;charset=UTF-8</value>
                        </list>
                    </property>
                    <property name="objectMapper">
                        <bean class="org.codehaus.jackson.map.ObjectMapper">
                            <property name="serializationInclusion" value="NON_NULL"/>
                        </bean>
                    </property>
                </bean>
            </mvc:message-converters>
        </mvc:annotation-driven>

#### <mvc:default-servlet-handler />
项目根目录下的index.jsp不生效，index.jsp是纯文本输出

解决方案与说明：在controller的配置文件中添加如下，告诉spring mvc将静态资源的处理交回Web应用服务器tomcat处理。(spring mvc主要是使用restful风格的)

#### pathVariable

    @RequestMapping(value="/comment/{blogId}", method=RequestMethod.POST)
    public void comment(@PathVariable int blogId, HttpSession session) {
    }

在该例子中，blogId是被@PathVariable标记为请求路径变量的，如果请求的是/blog/comment/1.do的时候就表示blogId的值为1

同样@RequestParam也是用来给参数传值的，但是它是从头request的参数里面取值，相当于request.getParameter("参数名")方法。
它的取值规则跟@PathVariable是一样的，当没有指定的时候，默认是从request中取名称跟后面接的变量名同名的参数值，
当要明确从request中取一个参数的时候使用@RequestParam("参数名")

#### @Scope("prototype")

    @Component("QueryMessageTask")
    @Scope("prototype")
    public class QueryMessageTask  implements Callable<List<TaskInfo>>
    Scope默认是singleton(单例).

#### map属性

    <!--邮件模板-->
    <bean id="velocityEngine"
          class="org.springframework.ui.velocity.VelocityEngineFactoryBean">
        <property name="resourceLoaderPath">
            <value>classpath:</value>
        </property>
        <property name="velocityProperties">
            <props>
                <prop key="input.encoding">UTF-8</prop>
                <prop key="output.encoding">UTF-8</prop>
                <prop key="contentType">text/html;charset=UTF-8</prop>
                <prop key="resource.loader">class</prop>
                <prop key="class.resource.loader.class">
                    org.apache.velocity.runtime.resource.loader.ClasspathResourceLoader
                </prop>
                <prop key="file.resource.loader.cache">false</prop>
                <prop key="file.resource.loader.modificationCheckInterval">1</prop>
                <prop key="velocimacro.library.autoreload">true</prop>
                <prop key="velocity.engine.resource.manager.cache.enabled">false</prop>
                <prop key="springMacro.resource.loader.cache">false</prop>
                <prop key="eventhandler.referenceinsertion.class">
                    org.apache.velocity.app.event.implement.EscapeXmlReference
                </prop>
            </props>
        </property>
    </bean>

### bean的生命周期

![bean的生命周期](/images/java_web/spring_beans_lifecycle.jpg)

- spring对bean进行实例化
- spring将值和bean的引用注入到bean对应的属性中
- 如果bean实现了BeanNameAware接口，spring将bean的ID传递给setBeanName()方法。
- 如果bean实现了BeanFactoryAware接口，spring将调用setBeanFactory()方法，将BeanFacotry容器实例传入
- 如果bean实现了ApplicationContextAware接口，spring将调用setApplicationContext()方法，将bean所在的应用上下文的引用传入进来
- 如果bean实现了BeanPostProcessor接口，spring将调用他们的postProcessBeforeInitialization()方法
- 如果bean实现了InitializationBean接口，spring将调用他们的afterPropertiesSet()方法，类似的，如果bean使用init-method声明了初始化方法，该方法也会被调用
- 如果bean实现了BeanPostProcessor接口，spring将调用他们的postProcessorAfterInitialization()方法
- 此时，bean已经准备就绪，可以被应用程序使用了，它们将一直驻留在应用上下文中，直到该应用上下文被销毁
- 如果bean实现了disposableBean接口，spring将调用它的destory()接口方法，同样，如果bean使用destory-method声明了销毁方法，该方法也会被调用

aware接口 -> 将单向关联转化为双向关联（Change Unidirectional Association to Bidirectional），详见[重构](/2017/05/13/refactor_list)

### component-scan  vs  annotation-config

```
1.<context:component-scan base-package="com.xxx">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
----------
2.<context:annotation-config/>
```

关于spring-bean的扫描有两个配置需要说下：

- context:component-scan

  1. 扫描注解@Component、@Service等实例化bean，
  2. 扫描注解@Resource、@Autowired等，管理依赖

- context:annotation-config

  1. 扫描注解@Resource、@Autowired等，管理依赖

  也就是说声明了context:annotation-config，还需要在xml中显式的声明bean

> `<context:annotation-config>` is used to activate annotations in beans already registered in the application context (no matter if they were defined with XML or by package scanning).

> `<context:component-scan>` can also do what `<context:annotation-config>` does but `<context:component-scan>` also scans packages to find and register beans within the application context.

> both tags register the same processing tools (`<context:annotation-config />` can be omitted if `<context:component-scan>` is specified) but Spring takes care of running them only once.

> I found this nice summary of which annotations are picked up by which declarations. By studying it you will find that `<context:component-scan/>` recognizes a superset of annotations recognized by `<context:annotation-config/>`, namely: @Component, @Service, @Repository, @Controller, @Endpoint @Configuration, @Bean, @Lazy, @Scope, @Order, @Primary, @Profile, @DependsOn, @Import, @ImportResource. As you can see

> <context:component-scan> logically extends <context:annotation-config> with CLASSPATH component scanning and Java @Configuration features.</context:annotation-config></context:component-scan>


ApplicationListener<ContextRefreshEvent>

Constructor > @PostConstruct > InitializingBean > init-method，构造函数最优先

# 参考

[component-scan-vs-annotation-config](http://stackoverflow.com/questions/7414794/difference-between-contextannotation-config-vs-contextcomponent-scan)
[scope](http://stackoverflow.com/questions/7621920/scopeprototype-bean-scope-not-creating-new-bean)
