---
layout: post
title: spring ioc启动流程
categories: java_web
tags: spring ioc ServletContext
---




本文基于spring4.3.1

### 入口-ContextLoaderListener

    <?xml version="1.1" encoding="UTF-8"?>
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://java.sun.com/xml/ns/javaee"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
    <display-name>xx Web Application</display-name>
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath*:root-context.xml</param-value>
    </context-param>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

对于互联网的java后台开发来说，这段配置应该都熟悉。

web.xml中的启动遵循context-param -> listener -> filter -> servlet。spring ioc的容器启动就是从ContextLoaderListener开始的。

    public class ContextLoaderListener extends ContextLoader implements ServletContextListener

关于ContextLoaderListener

1. 继承自`org.springframework.web.context.ContextLoader`，具备load context的能力，这个context是spring的ioc容器。
2. 实现了`javax.servlet.ServletContextListener`,监听ServletContext的event(init和destory)
   * contextInitialized 在contextInitialized方法中load context(对于XmlWebApplicationContext是initWebApplicationContext方法)
   * contextDestroyed
3. 三种context
   * ServletContext，web容器，可以是tomcat，可以是jetty等
   * Root WebApplicationContext，spring的ioc容器
   * 如果使用了spring mvc，还会有DispatcherServlet对应的webApplicationContext。

下是关于ServletContext的官方描述。

    /**
     * Defines a set of methods that a servlet uses to communicate with its servlet
     * container, for example, to get the MIME type of a file, dispatch requests, or
     * write to a log file.
     *
     * There is one context per "web application" per Java Virtual Machine. (A
     * "web application" is a collection of servlets and content installed under a
     * specific subset of the server's URL namespace such as /catalog
     * and possibly installed via a .war file.)
     *
     * In the case of a web application marked "distributed" in its deployment
     * descriptor, there will be one context instance for each virtual machine. In
     * this situation, the context cannot be used as a location to share global
     * information (because the information won't be truly global). Use an external
     * resource like a database instead.
     *
     * The ServletContext object is contained within the
     *{@link ServletConfig}object, which the Web server provides the servlet when
     * the servlet is initialized.
     */

本文只讨论Servletcontext和Root WebApplicationContext.  dispatcherServlet的context不在本文讨论范围之内。

以web应用使用的WebXmlWebApplicationContext作为切入点分析spring ioc的建立流程，ClassPathXmlWebApplicationContext、FileXmlWebApplicationContext类似。

### XmlWebApplicationContext类图

首先我们先看下其类图，通过其实现的接口看下它具备的能力。

![xmlWebApplicationContext类图](/images/java_web/xmlwc_uml.png)

通过类图，XmlWebApplicationContext实现的接口，我们可以看出其具备的行为：

1. ResourceLoader: 具备加载资源的能力

2. BeanFactory: ioc容器的最顶层接口，获取容器中的bean。HierarchicalBeanFactory表明其是有层级结构的，ListableBeanFactory表明其是可以列举其beans的，其实现类会预加载其所有的beanDefinitions的

3. ...

对应的其处理流程：

1. 加载resource, 由context-param标签中的参数指定。

2. 解析resource，向容器中注册resspringource对应的bean,即填充beanDefinitionMap等bean的元信息

3. 依据BeanDefinition，实例化bean(lazy-init = false或者在第一次使用bean时)

4. bean定义的载入和依赖注入是两个独立的过程。

### 启动流程

下边这个方法算是ioc容器启动的主体方法了。里边的每个方法都对应一大堆逻辑，我们捡几个重要的，能反映主体流程的说下。

    @Override
    public void refresh() throws BeansException, IllegalStateException {
        synchronized (this.startupShutdownMonitor) {
            // Prepare this context for refreshing.
            prepareRefresh();

            // Tell the subclass to refresh the internal bean factory.
            // 加载resource，解析Resource，向beanFactory中填充BeanDefinition等bean的元信息。
            ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

            // Prepare the bean factory for use in this context.
            prepareBeanFactory(beanFactory);

            try {
                // Allows post-processing of the bean factory in context subclasses.
                postProcessBeanFactory(beanFactory);

                // Invoke factory processors registered as beans in the context.
                invokeBeanFactoryPostProcessors(beanFactory);

                // Register bean processors that intercept bean creation.
                registerBeanPostProcessors(beanFactory);

                // Initialize message source for this context.
                initMessageSource();

                // Initialize event multicaster for this context.
                initApplicationEventMulticaster();

                // Initialize other special beans in specific context subclasses.
                onRefresh();

                // Check for listener beans and register them.
                registerListeners();

                // Instantiate all remaining (non-lazy-init) singletons.
                // 实例化非lazy-init的单例。顺带着也会实例化其依赖的bean
                finishBeanFactoryInitialization(beanFactory);

                // Last step: publish corresponding event.
                finishRefresh();
            }

            catch (BeansException ex) {
                if (logger.isWarnEnabled()) {
                    logger.warn("Exception encountered during context initialization - " +
                            "cancelling refresh attempt: " + ex);
                }

                // Destroy already created singletons to avoid dangling resources.
                destroyBeans();

                // Reset 'active' flag.
                cancelRefresh(ex);

                // Propagate exception to caller.
                throw ex;
            }

            finally {
                // Reset common introspection caches in Spring's core, since we
                // might not ever need metadata for singleton beans anymore...
                resetCommonCaches();
            }
        }
    }

#### obtainFreshBeanFactory

实例化bean有两种方式：xml中配bean或者扫注解。以component-scan扫注解的为例

首先看下其调用栈，![beandefinitionMap](/images/java_web/ioc_call_stack.png)

1. 从调用栈可以看出，从入口ContextLoaderListener到扫描（scan）涉及的对象有：
    *   `ContextLoaderListener`、
    *   `ContextLoader`
    *   `AbstractApplicationContext`、
    *   `AbstractRefreshableApplicationContext`、
    *   `XmlWebApplicationContext`、
    *   `AbstractBeanDefinitionReader`、
    *   `XmlBeanDefinitionReader`、
    *   `DefaultBeanDefinitionDocumentReader`、
    *   `BeanDefinitionParserDelegate`、
    *   `NamespaceHandlerSupport`、
    *   `ComponentScanBeanDefinitionParser`、
    *   `ClassPathBeanDefinitionScanner`
2. DefaultBeanDefinitionDocumentReader#parseDefaultElement: 默认namespace的解析，（xml） 解析bean、import、beans、alias标签
3. BeanDefinitionParserDelegate#parseCustomElement: 其他namespace 的解析 （注解），如tx、context等
   * 每个namespace对应有不同的`NamespaceHandler`，eg :tx、context
   * 每个namespaceHandler可能对应多个`BeanDefinitionParser`,eg: ContextNamespaceHandler中包含有PropertyPlaceholderBeanDefinitionParser、ComponentScanBeanDefinitionParser、AnnotationConfigBeanDefinitionParser等用于处理context namespace下的不同的标签。
具体的处理逻辑可以看对应的parser。
4. spring通过定义BeanDefinition来管理基于spring的应用中的各种对象以及它们之间的相互依赖关系。

##### DefaultBeanDefinitionDocumentReader#parseDefaultElement

默认namespace的解析，bean、beans、import、alias

##### BeanDefinitionParserDelegate#parseCustomElement

其他namespace的解析，如tx，context等

下是component-scan扫注解，加载bean配置的时序图（最好打开个新连接看，图太小了）
![spring_ioc_bean_definition](/images/java_web/spring_ioc_beanDefinitions.jpg)


#### finishBeanFactoryInitialization

我们只说关键方法，从doCreateBean说明，其调用链如下：

    "RMI TCP Connection(2)-127.0.0.1@1870" daemon prio=5 tid=0x15 nid=NA runnable
      java.lang.Thread.State: RUNNABLE
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:525)
          at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:482)
          at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
          at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
          - locked <0xe5d> (a java.util.concurrent.ConcurrentHashMap)
          at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
          at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
          at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:775)
          at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:861)
          at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:541)

关键方法如下：

##### AbstractAutowireCapableBeanFactory#createBean

实例化bean，包括bean初始化的前置后置逻辑，包括bean属性的填充

    @Override
    protected Object createBean(String beanName, RootBeanDefinition mbd, Object[] args) throws BeanCreationException {
        if (logger.isDebugEnabled()) {
            logger.debug("Creating instance of bean '" + beanName + "'");
        }
        RootBeanDefinition mbdToUse = mbd;

        // Make sure bean class is actually resolved at this point, and
        // clone the bean definition in case of a dynamically resolved Class
        // which cannot be stored in the shared merged bean definition.
        Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
        if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
            mbdToUse = new RootBeanDefinition(mbd);
            mbdToUse.setBeanClass(resolvedClass);
        }

        // Prepare method overrides.
        try {
            mbdToUse.prepareMethodOverrides();
        }
        catch (BeanDefinitionValidationException ex) {
            throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
                    beanName, "Validation of method overrides failed", ex);
        }

        try {
            // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
            // 拦截逻辑
            Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
            if (bean != null) {
                return bean;
            }
        }
        catch (Throwable ex) {
            throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
                    "BeanPostProcessor before instantiation of bean failed", ex);
        }
        // 实例化
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        if (logger.isDebugEnabled()) {
            logger.debug("Finished creating instance of bean '" + beanName + "'");
        }
        return beanInstance;
    }

##### AbstractAutowireCapableBeanFactory#doCreateBean

    protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args) {
        // Instantiate the bean.
        BeanWrapper instanceWrapper = null;
        if (mbd.isSingleton()) {
            instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
        }
        if (instanceWrapper == null) {
            // 构造bean
            instanceWrapper = createBeanInstance(beanName, mbd, args);
        }
        final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
        Class<?> beanType = (instanceWrapper != null ? instanceWrapper.getWrappedClass() : null);

        // Allow post-processors to modify the merged bean definition.
        synchronized (mbd.postProcessingLock) {
            if (!mbd.postProcessed) {
                // 处理beandefinition的各种后置拦截逻辑，
                // 包括AutowiredAnnotationBeanPostProcessor、CommonAnnotationBeanPostProcessor解析@Resource和@Autowired等标签
                applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
                mbd.postProcessed = true;
            }
        }

        // Eagerly cache singletons to be able to resolve circular references
        // even when triggered by lifecycle interfaces like BeanFactoryAware.
        boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
                isSingletonCurrentlyInCreation(beanName));
        if (earlySingletonExposure) {
            if (logger.isDebugEnabled()) {
                logger.debug("Eagerly caching bean '" + beanName +
                        "' to allow for resolving potential circular references");
            }
            addSingletonFactory(beanName, new ObjectFactory<Object>() {
                @Override
                public Object getObject() throws BeansException {
                    return getEarlyBeanReference(beanName, mbd, bean);
                }
            });
        }

        // Initialize the bean instance.
        Object exposedObject = bean;
        try {
            // 填充bean，解析@Resource、@Autowired等标签
            populateBean(beanName, mbd, instanceWrapper);
            if (exposedObject != null) {
                // 实例化属性（可能需要代理）
                exposedObject = initializeBean(beanName, exposedObject, mbd);
            }
        }
        catch (Throwable ex) {
            if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
                throw (BeanCreationException) ex;
            }
            else {
                throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
            }
        }

        if (earlySingletonExposure) {
            Object earlySingletonReference = getSingleton(beanName, false);
            if (earlySingletonReference != null) {
                if (exposedObject == bean) {
                    exposedObject = earlySingletonReference;
                }
                else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
                    String[] dependentBeans = getDependentBeans(beanName);
                    Set<String> actualDependentBeans = new LinkedHashSet<String>(dependentBeans.length);
                    for (String dependentBean : dependentBeans) {
                        if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                            actualDependentBeans.add(dependentBean);
                        }
                    }
                    if (!actualDependentBeans.isEmpty()) {
                        throw new BeanCurrentlyInCreationException(beanName,
                                "Bean with name '" + beanName + "' has been injected into other beans [" +
                                StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
                                "] in its raw version as part of a circular reference, but has eventually been " +
                                "wrapped. This means that said other beans do not use the final version of the " +
                                "bean. This is often the result of over-eager type matching - consider using " +
                                "'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
                    }
                }
            }
        }

        // Register bean as disposable.
        try {
            registerDisposableBeanIfNecessary(beanName, bean, mbd);
        }
        catch (BeanDefinitionValidationException ex) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
        }

        return exposedObject;
    }


##### AbstractAutowireCapableBeanFactory#populateBean

解析依赖，填充属性（深度优先）

##### AbstractAutowireCapableBeanFactory#initializeBean

初始化bean，实例化对应属性的bean，可能需要创建代理，实例化策略：

* 无代理实例化，
* cglib
* jdk动态代理

调用栈如下：


       getProxy:164, CglibAopProxy {org.springframework.aop.framework}
       getProxy:109, ProxyFactory {org.springframework.aop.framework}
       createProxy:470, AbstractAutoProxyCreator {org.springframework.aop.framework.autoproxy}
       wrapIfNecessary:350, AbstractAutoProxyCreator {org.springframework.aop.framework.autoproxy}
       postProcessAfterInitialization:299, AbstractAutoProxyCreator {org.springframework.aop.framework.autoproxy}
       applyBeanPostProcessorsAfterInitialization:422, AbstractAutowireCapableBeanFactory {org.springframework.beans.factory.support}
       initializeBean:1583, AbstractAutowireCapableBeanFactory {org.springframework.beans.factory.support}
       doCreateBean:545, AbstractAutowireCapableBeanFactory {org.springframework.beans.factory.support}
       createBean:482, AbstractAutowireCapableBeanFactory {org.springframework.beans.factory.support}
       getObject:306, AbstractBeanFactory$1 {org.springframework.beans.factory.support}
       getSingleton:230, DefaultSingletonBeanRegistry {org.springframework.beans.factory.support}
       doGetBean:302, AbstractBeanFactory {org.springframework.beans.factory.support}
       getBean:197, AbstractBeanFactory {org.springframework.beans.factory.support}
       preInstantiateSingletons:775, DefaultListableBeanFactory {org.springframework.beans.factory.support}
       finishBeanFactoryInitialization:861, AbstractApplicationContext {org.springframework.context.support}
       refresh:541, AbstractApplicationContext {org.springframework.context.support}
       configureAndRefreshWebApplicationContext:444, ContextLoader {org.springframework.web.context}
       initWebApplicationContext:326, ContextLoader {org.springframework.web.context}
       contextInitialized:107, ContextLoaderListener {org.springframework.web.context}

参考：

[1]<http://blog.csdn.net/c289054531/article/details/9196149>

[2]<https://www.ibm.com/developerworks/cn/java/j-lo-servlet/>

[Spring MVC注解故障追踪记](http://tech.meituan.com/mt-trip-springmvc-service-annotation-problem-research.html)

[看看Spring的源码（一）——Bean加载过程](http://geeekr.com/read-spring-source-1-how-to-load-bean/)
