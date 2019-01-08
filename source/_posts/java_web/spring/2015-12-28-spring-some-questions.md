---
layout: post
title: 关于spring的几个问题
categories: spring
tags:
---




几个开发中可能会遇到的问题。

本文都是先说结论，然后以源码做注解。

前提:基于spring 4.3.1,使用XmlWebApplicationContext,使用默认策略

#### xml声明的bean和注解声明的bean的覆盖规则是怎样的？

1. **同一容器内，xml会覆盖之前声明的同beanName的beanDefinition（包括注解和xml）**
2. **同一容器内，注解不会覆盖之前声明的同beanName的beanDefinition**

源码注解：

1.`ClassPathBeanDefinitionScanner#checkCandidate`

2.`DefaultListableBeanFactory#registerBeanDefinition`

    // 返回是否可以被扫描
    protected boolean checkCandidate(String beanName, BeanDefinition beanDefinition) throws IllegalStateException {
        // 1.如果注册表中不存在该beanName，返回true
        if (!this.registry.containsBeanDefinition(beanName)) {
            return true;
        }
        // 2.通过beanName找到对应的bean
        BeanDefinition existingDef = this.registry.getBeanDefinition(beanName);
        BeanDefinition originatingDef = existingDef.getOriginatingBeanDefinition();
        if (originatingDef != null) {
        	existingDef = originatingDef;
        }
        // 3.是否兼容，如果存在的beanDefinition兼容要被扫描的beanDefinition，则没必要再次被扫描，然后我们看下isCompatible的逻辑
        if (isCompatible(beanDefinition, existingDef)) {
        	return false;
        }
        throw new ConflictingBeanDefinitionException("Annotation-specified bean name '" + beanName +
                "' for bean class [" + beanDefinition.getBeanClassName() + "] conflicts with existing, " +
                    "non-compatible bean definition of same name and class [" + existingDef.getBeanClassName() + "]");
	}

    // 1. 已经存在的beanDefinition不是ScannedGenericBeanDefinition，显然xml声明的bean不是scanned的
    // 2. 已经存在的beanDefinition和new 的beanDefinition是同一source
    // 3. 两个beanDefinition equals
    // 三者条件满足其一，compatible都是true,那么都不需要再被扫描，所以xml声明过的bean是不会再被扫描的
    protected boolean isCompatible(BeanDefinition newDefinition, BeanDefinition existingDefinition) {
        return (!(existingDefinition instanceof ScannedGenericBeanDefinition) ||  // explicitly registered overriding bean
                newDefinition.getSource().equals(existingDefinition.getSource()) ||  // scanned same file twice
                newDefinition.equals(existingDefinition));  // scanned equivalent class twice
    }

    public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
    			throws BeanDefinitionStoreException {
        Assert.hasText(beanName, "Bean name must not be empty");
        Assert.notNull(beanDefinition, "BeanDefinition must not be null");
        if (beanDefinition instanceof AbstractBeanDefinition) {
            try {
                ((AbstractBeanDefinition) beanDefinition).validate();
            }
            catch (BeanDefinitionValidationException ex) {
                throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
                    "Validation of bean definition failed", ex);
            }
        }
        synchronized (this.beanDefinitionMap) {
            Object oldBeanDefinition = this.beanDefinitionMap.get(beanName);
            if (oldBeanDefinition != null) {
                // 对于XmlWebApplicationContext使用的DefaultListableBeanFactory，是允许覆盖的，allowBeanDefinitionOverriding为true
                if (!this.allowBeanDefinitionOverriding) {
                    throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
                        "Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +
                            "': There is already [" + oldBeanDefinition + "] bound.");
                }
                else {
                    if (this.logger.isInfoEnabled()) {
                    	this.logger.info("Overriding bean definition for bean '" + beanName +
                            "': replacing [" + oldBeanDefinition + "] with [" + beanDefinition + "]");
                    }
                }
            }
        	else {
                this.beanDefinitionNames.add(beanName);
                this.frozenBeanDefinitionNames = null;
        	}
            // 这里xml会覆盖之前声明的同beanName的beanDefinition(当然也包括注解声明的bean)
        	this.beanDefinitionMap.put(beanName, beanDefinition);
        }
        resetBeanDefinition(beanName);
    }

#### contextConfigLocation

首先上一段配置：

    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns="http://java.sun.com/xml/ns/j2ee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
         version="2.4">
    <display-name>meilv WebApp</display-name>

    <!-- context-params-->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:applicationContext.xml</param-value>
        <!--<param-value>classpath*:spring/**/spring-*.xml</param-value>-->
        <!--<param-value>root-context.xml</param-value>-->
    </context-param>

    ...
    </web-app>

其中context-param的contextConfigLocation的value可以有三种形式：
classpath*、classpath、不带classpath的,然后又可细分为antPath和指明的文件，如下

1. classpath*: all class path resources，包含依赖的jar包
    1. **a class path resource pattern**, classpath下满足给定antpath的所有文件（包含jar包中的）
    2. **all class path resources with the given name**,classpath下文件名为给定的名字的所有文件（包含jar包中的）
2. classpath: 不会找依赖的jar包里的resource，和classpath*类似，不过不包含jar包中的
    1. **a file pattern**
    2. **a single resource with the given name**
3. 普通文件:会从webApp的`/WEB-INF/`下找
    1. **a file pattern**
    2. **a single resource with the given name**

ps: war包的结构

    .
    |____index.jsp
    |____META-INF
    |____WEB-INF
    | |____classes
    | |____lib
    | |____web.xml

源码注解：

1.`PathMatchingResourcePatternResolver#getResources`

2.`PathMatchingResourcePatternResolver#findPathMatchingResources`

3.`PathMatchingResourcePatternResolver#findAllClassPathResources`

4.`DefaultResourceLoader#getResource`

    @Override
    public Resource[] getResources(String locationPattern) throws IOException {
        Assert.notNull(locationPattern, "Location pattern must not be null");
        // 如果以classpath*:开头
        if (locationPattern.startsWith(CLASSPATH_ALL_URL_PREFIX)) {
            // a class path resource (multiple resources for same name possible)
            // 默认的getPathMatcher 都是AntPathMatcher，
            // 如果除去classpath*:外还包含有*或者?字符，走findPathMatchingResources
            if (getPathMatcher().isPattern(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()))) {
                // a class path resource pattern
                return findPathMatchingResources(locationPattern);
            }
            else {
                // all class path resources with the given name
                return findAllClassPathResources(locationPattern.substring(CLASSPATH_ALL_URL_PREFIX.length()));
            }
        }
        else {
            // Only look for a pattern after a prefix here
            // (to not get fooled by a pattern symbol in a strange prefix).
            int prefixEnd = locationPattern.indexOf(":") + 1;
            if (getPathMatcher().isPattern(locationPattern.substring(prefixEnd))) {
                // a file pattern
                return findPathMatchingResources(locationPattern);
            }
            else {
                // a single resource with the given name
                return new Resource[] {getResourceLoader().getResource(locationPattern)};
            }
        }
    }

    protected Resource[] findPathMatchingResources(String locationPattern) throws IOException {
        // 第一个/之前的路径为rootDirPath
        String rootDirPath = determineRootDir(locationPattern);
        // rootDirPath之后的路径为subPattern
        String subPattern = locationPattern.substring(rootDirPath.length());
        // eg: 对于classpath*:spring/**/spring-*.xml，其rootDirPath为classpath*:spring/，subPattern为**/spring-*.xml
        Resource[] rootDirResources = getResources(rootDirPath);
        Set<Resource> result = new LinkedHashSet<Resource>(16);
        for (Resource rootDirResource : rootDirResources) {
        	rootDirResource = resolveRootDirResource(rootDirResource);
        	URL rootDirURL = rootDirResource.getURL();
        	if (equinoxResolveMethod != null) {
        		if (rootDirURL.getProtocol().startsWith("bundle")) {
        			rootDirURL = (URL) ReflectionUtils.invokeMethod(equinoxResolveMethod, null, rootDirURL);
        			rootDirResource = new UrlResource(rootDirURL);
        		}
        	}
        	if (rootDirURL.getProtocol().startsWith(ResourceUtils.URL_PROTOCOL_VFS)) {
        		result.addAll(VfsResourceMatchingDelegate.findMatchingResources(rootDirURL, subPattern, getPathMatcher()));
        	}
        	else if (ResourceUtils.isJarURL(rootDirURL) || isJarResource(rootDirResource)) {
        		result.addAll(doFindPathMatchingJarResources(rootDirResource, rootDirURL, subPattern));
        	}
        	else {
        		result.addAll(doFindPathMatchingFileResources(rootDirResource, subPattern));
        	}
        }
        if (logger.isDebugEnabled()) {
        	logger.debug("Resolved location pattern [" + locationPattern + "] to resources " + result);
        }
        return result.toArray(new Resource[result.size()]);
	}

    protected Resource[] findAllClassPathResources(String location) throws IOException {
        String path = location;
        if (path.startsWith("/")) {
            path = path.substring(1);
        }
        Set<Resource> result = doFindAllClassPathResources(path);
        if (logger.isDebugEnabled()) {
            logger.debug("Resolved classpath location [" + location + "] to resources " + result);
        }
        return result.toArray(new Resource[result.size()]);
    }

#### beanName、id

1. 对于xml中声明的bean，如果指定了id属性，beanName为id对应的值，如果没有指定id属性，则spring会自己generate个beanName，规则为:`className#数字`

2. 对于注解声明的bean，如果指定了value，则beanName使用指定的value；如果没指定，则spring会自己generate个beanName，规则为：simpleClassName，然后首字母小写（如果前两个字符都是大写的话，就直接返回simpleClassName了）

源码注解：

xml：

    public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
        // 解析id属性
        String id = ele.getAttribute(ID_ATTRIBUTE);
        // 解析name属性
        String nameAttr = ele.getAttribute(NAME_ATTRIBUTE);

        List<String> aliases = new ArrayList<String>();
        if (StringUtils.hasLength(nameAttr)) {
            String[] nameArr = StringUtils.tokenizeToStringArray(nameAttr, MULTI_VALUE_ATTRIBUTE_DELIMITERS);
            aliases.addAll(Arrays.asList(nameArr));
        }

        // spring容器中会使用id作为该beanDefinition的beanName
        String beanName = id;
        if (!StringUtils.hasText(beanName) && !aliases.isEmpty()) {
            beanName = aliases.remove(0);
            if (logger.isDebugEnabled()) {
                logger.debug("No XML 'id' specified - using '" + beanName +
                        "' as bean name and " + aliases + " as aliases");
            }
        }

        if (containingBean == null) {
            checkNameUniqueness(beanName, aliases, ele);
        }
        // 对标签其他属性的解析过程
        AbstractBeanDefinition beanDefinition = parseBeanDefinitionElement(ele, beanName, containingBean);
        if (beanDefinition != null) {
            if (!StringUtils.hasText(beanName)) {
                try {
                    // 如果不存在beanName，spring为当前bean生成对应的beanName
                    if (containingBean != null) {
                        beanName = BeanDefinitionReaderUtils.generateBeanName(
                                beanDefinition, this.readerContext.getRegistry(), true);
                    }
                    else {
                        beanName = this.readerContext.generateBeanName(beanDefinition);
                        // Register an alias for the plain bean class name, if still possible,
                        // if the generator returned the class name plus a suffix.
                        // This is expected for Spring 1.2/2.0 backwards compatibility.
                        String beanClassName = beanDefinition.getBeanClassName();
                        if (beanClassName != null &&
                                beanName.startsWith(beanClassName) && beanName.length() > beanClassName.length() &&
                                !this.readerContext.getRegistry().isBeanNameInUse(beanClassName)) {
                            aliases.add(beanClassName);
                        }
                    }
                    if (logger.isDebugEnabled()) {
                        logger.debug("Neither XML 'id' nor 'name' specified - " +
                                "using generated bean name [" + beanName + "]");
                    }
                }
                catch (Exception ex) {
                    error(ex.getMessage(), ele);
                    return null;
                }
            }
            String[] aliasesArray = StringUtils.toStringArray(aliases);
            return new BeanDefinitionHolder(beanDefinition, beanName, aliasesArray);
        }

        return null;
    }

注解：

    protected String buildDefaultBeanName(BeanDefinition definition) {
        String shortClassName = ClassUtils.getShortName(definition.getBeanClassName());
        return Introspector.decapitalize(shortClassName);
    }

    public static String getShortName(String className) {
		Assert.hasLength(className, "Class name must not be empty");
		int lastDotIndex = className.lastIndexOf(PACKAGE_SEPARATOR);
		int nameEndIndex = className.indexOf(CGLIB_CLASS_SEPARATOR);
		if (nameEndIndex == -1) {
			nameEndIndex = className.length();
		}
		String shortName = className.substring(lastDotIndex + 1, nameEndIndex);
		shortName = shortName.replace(INNER_CLASS_SEPARATOR, PACKAGE_SEPARATOR);
		return shortName;
	}

    public static String decapitalize(String name) {
        if (name == null || name.length() == 0) {
            return name;
        }
        if (name.length() > 1 && Character.isUpperCase(name.charAt(1)) &&
                        Character.isUpperCase(name.charAt(0))){
            return name;
        }
        char chars[] = name.toCharArray();
        chars[0] = Character.toLowerCase(chars[0]);
        return new String(chars);
    }


#### @Resource和@Autowired的区别

AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInstantiation

InstantiationAwareBeanPostProcessor

[spring中Autowired和Resource的区别](http://qiangmzsx.blog.51cto.com/2052549/1359952)

#### 动态代理的几个问题

    "RMI TCP Connection(2)-127.0.0.1@1974" daemon prio=5 tid=0x15 nid=NA runnable
      java.lang.Thread.State: RUNNABLE
    	  at org.springframework.cglib.core.DefaultNamingPolicy.getClassName(DefaultNamingPolicy.java:39)
    	  at org.springframework.cglib.core.AbstractClassGenerator.generateClassName(AbstractClassGenerator.java:154)
    	  at org.springframework.cglib.core.AbstractClassGenerator.generate(AbstractClassGenerator.java:317)
    	  - locked <0xdf7> (a org.apache.catalina.loader.WebappClassLoader)
    	  at org.springframework.cglib.proxy.Enhancer.generate(Enhancer.java:492)
    	  at org.springframework.cglib.core.AbstractClassGenerator$ClassLoaderData$3.apply(AbstractClassGenerator.java:93)
    	  at org.springframework.cglib.core.AbstractClassGenerator$ClassLoaderData$3.apply(AbstractClassGenerator.java:91)
    	  at org.springframework.cglib.core.internal.LoadingCache$2.call(LoadingCache.java:54)
    	  at java.util.concurrent.FutureTask.run(FutureTask.java:266)
    	  at org.springframework.cglib.core.internal.LoadingCache.createEntry(LoadingCache.java:61)
    	  at org.springframework.cglib.core.internal.LoadingCache.get(LoadingCache.java:34)
    	  at org.springframework.cglib.core.AbstractClassGenerator$ClassLoaderData.get(AbstractClassGenerator.java:116)
    	  at org.springframework.cglib.core.AbstractClassGenerator.create(AbstractClassGenerator.java:291)
    	  at org.springframework.cglib.proxy.Enhancer.createHelper(Enhancer.java:480)
    	  at org.springframework.cglib.proxy.Enhancer.createClass(Enhancer.java:337)
    	  at org.springframework.aop.framework.ObjenesisCglibAopProxy.createProxyClassAndInstance(ObjenesisCglibAopProxy.java:55)
    	  at org.springframework.aop.framework.CglibAopProxy.getProxy(CglibAopProxy.java:203)
    	  at org.springframework.aop.framework.ProxyFactory.getProxy(ProxyFactory.java:109)
    	  at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.createProxy(AbstractAutoProxyCreator.java:470)
    	  at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.wrapIfNecessary(AbstractAutoProxyCreator.java:350)
    	  at org.springframework.aop.framework.autoproxy.AbstractAutoProxyCreator.postProcessAfterInitialization(AbstractAutoProxyCreator.java:299)
    	  at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsAfterInitialization(AbstractAutowireCapableBeanFactory.java:422)
    	  at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1583)
    	  at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:545)
    	  at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:482)
    	  at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:306)
    	  at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:230)
    	  - locked <0xeaf> (a java.util.concurrent.ConcurrentHashMap)
    	  at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:302)
    	  at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:197)
    	  at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:775)
    	  at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:861)
    	  at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:541)
    	  - locked <0x1270> (a java.lang.Object)

##### 代理的时机

##### 代理的种类

#### spring的回调接口

参考：

[spring4.0 源码分析 bean标签的解析(三)](http://blog.csdn.net/sun_aichao/article/details/50284715)

[spring-alias 标签的解析](http://www.cnblogs.com/mjorcen/p/3649548.html)
