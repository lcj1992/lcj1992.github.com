---
layout: post
title: clean pom
date: 2015-12-28
categories: soft
tags:
    - maven
---

### 基础知识

1. 依赖调解 dependency mediation

   * nearest  A->B->C2; A->C1   最终会选择C1
   * 声明优先 A->C2; B->C1,   最终会选择C2

2. 依赖管理 dependency management  

   * 可以显式指定，来解决多版本问题（传递依赖）。

3. 依赖作用域

   * compile、test、provided、system、runtime、import

4. 排除依赖 excluded dependencies

5. 可选依赖 optional dependencies   

  * X-> A  A->B 可以将B声明为optional，这样X就不会依赖B，如果X想引入B依赖，需显式声明,eg: [可选例子](http://ifeve.com/introduction-to-optional-and-excludes-dependencies/)

> 有一个名为X2的项目，这个项目和hibernate有一些类似的功能，支持许多数据库驱动/依赖，比如说MySQL，postgre，oracle等。为了构建X2，所有的这些依赖都是必须的，但是对于你的项目来说却不是必须的，所以对于X2把这些依赖声明为可选的是非常实用的，不论什么时候当你在POM文件中把X2声明为一个直接依赖的时候，所有被X2支持的驱动不会自动的被包含进你的项目的classpath，你需要直接声明你将要使用的数据库的依赖/驱动。

6. 优先级

  * 依赖管理显式引入的优先级高于依赖调解机制调解出来的
  * 当前pom的声明优先于父pom中的声明（对于dependency的，区分是groupId:artifactId:type:classifier，四个都相同是前提；对于plugin是groupId和artifactId），详见下代码

>1. dependency management takes precedence over dependency mediation for transitive dependencies
>2. the current pom's declaration takes precedence over its parent's declaration.

#### 子pom的dependencies和父pom的dependencies如何merge

包括dependencyManagement中的dependencies和真正的dependencies，以源码做注解[maven github地址](https://github.com/apache/maven)

    // target：这里是子pom的dependencyManagement
    // source：这里是父pom的dependencyManagement
    // sourceDominant：是否source占主导，这里是false
    protected void mergeDependencyManagement_Dependencies( DependencyManagement target, DependencyManagement source,
                                                           boolean sourceDominant, Map<Object, Object> context )
    {
        List<Dependency> src = source.getDependencies();
        if ( !src.isEmpty() )
        {
            List<Dependency> tgt = target.getDependencies();
            Map<Object, Dependency> merged = new LinkedHashMap<>( ( src.size() + tgt.size() ) * 2 );


            // 注意dependency是包含excludes的。
            for ( Dependency element : tgt )
            {
                Object key = getDependencyKey( element );
                merged.put( key, element );
            }

            for ( Dependency element : src )
            {
                // key是groupId+artifactId+type+classifier组成的字符串，见下getManagementKey
                Object key = getDependencyKey( element );
                //  sourceDominant为false
                if ( sourceDominant || !merged.containsKey( key ) )
                {
                    merged.put( key, element );
                }
            }

            target.setDependencies( new ArrayList<>( merged.values() ) );
        }
    }

    @Override
    protected Object getDependencyKey( Dependency dependency )
    {
        return dependency.getManagementKey();
    }

    public String getManagementKey()
    {
        if ( managementKey == null )
        {
            managementKey = groupId + ":" + artifactId + ":" + type + ( classifier != null ? ":" + classifier : "" );
        }
        return managementKey;
    }

#### dependency如何使用dependencyManagement的配置

    public void mergeManagedDependencies( Model model )
    {
        DependencyManagement dependencyManagement = model.getDependencyManagement();
        if ( dependencyManagement != null )
        {
            Map<Object, Dependency> dependencies = new HashMap<>();
            Map<Object, Object> context = Collections.emptyMap();

            for ( Dependency dependency : model.getDependencies() )
            {
                // 同上 groupId:artifactId:type:classifier
                Object key = getDependencyKey( dependency );
                dependencies.put( key, dependency );
            }

            for ( Dependency managedDependency : dependencyManagement.getDependencies() )
            {
                Object key = getDependencyKey( managedDependency );
                Dependency dependency = dependencies.get( key );
                if ( dependency != null )
                {
                    mergeDependency( dependency, managedDependency, false, context );
                }
            }
        }
    }

    protected void mergeDependency( Dependency target, Dependency source, boolean sourceDominant,
                                    Map<Object, Object> context )
    {
        mergeDependency_GroupId( target, source, sourceDominant, context );
        mergeDependency_ArtifactId( target, source, sourceDominant, context );
        mergeDependency_Version( target, source, sourceDominant, context );
        mergeDependency_Type( target, source, sourceDominant, context );
        mergeDependency_Classifier( target, source, sourceDominant, context );
        mergeDependency_Scope( target, source, sourceDominant, context );
        mergeDependency_SystemPath( target, source, sourceDominant, context );
        mergeDependency_Optional( target, source, sourceDominant, context );
        mergeDependency_Exclusions( target, source, sourceDominant, context );
    }


    protected void mergeDependency_Version( Dependency target, Dependency source, boolean sourceDominant,
                                            Map<Object, Object> context )
    {
        String src = source.getVersion();
        if ( src != null )
        {
            // 如果dependency中没有声明版本，会取dependencyManagement的（groupId、artifactId、type、classifier、scope、systemPath、optional类似）
            if ( sourceDominant || target.getVersion() == null )
            {
                target.setVersion( src );
                target.setLocation( "version", source.getLocation( "version" ) );
            }
        }
    }

    @Override
    protected void mergeDependency_Exclusions( Dependency target, Dependency source, boolean sourceDominant,
                                               Map<Object, Object> context )
    {
        List<Exclusion> tgt = target.getExclusions();
        // 只有当tgt的exclusions为空时，才会取dependencyManagement中的exclusions
        if ( tgt.isEmpty() )
        {
            List<Exclusion> src = source.getExclusions();

            for ( Exclusion element : src )
            {
                Exclusion clone = element.clone();
                target.addExclusion( clone );
            }
        }
    }

### 最佳实践

1. 项目的parent设置为公共父pom（指定了一些每个项目都需要的配置，不需要你再重复了，包括jdk配置、plugin配置、仓库配置、常用jar的dependencyManagement；引入冲突检测机制，将风险在编译期就暴露出来）

2. 依赖冲突都在项目的父模块中解决，子模块中引入

3. 尽量使用依赖（dependencyManagement）解决冲突，而不是依赖调解exclude

4. 统一日志，使用slf4j + log4j2，详见日志相关

5. 线上只引release包，禁止引snapshot包。当然一点点来吧。

6. jar禁止包含工程配置文件：默认屏蔽的文件有:*.xml，*.properties

### 调试maven  

主要是想验证下上边一些不确定的地方，官网没找到相关内容，网上说的不可信，就自己debug源码看了。

    // 1. 下载maven源码
    git clone git@github.com:apache/maven.git

    // 2. build maven（你机器上得有maven和jdk）
    mvn -DdistributionTargetFolder="/path/to/maven" clean package

    // 3. 使用你源码build出的命令（不是你之前安装的哟），mvnDebug会设置java的debug端口为8000
    cd  /path/to/you_project
    /path/to/maven/bin/mvnDebug  clean package(你要执行的命令)

    // 4.设置idea的remote debug配置为localhost 8000，入口是MavenCli#main，

    // 5.调试吧。

### 参考

[maven依赖机制](https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html)

[maven](https://maven.apache.org/ref/3.3.9/maven-model/maven.html#class_DependencyManagement)

[Maven 工程规范](http://blog.csdn.net/oooyooo/article/details/49254571)
