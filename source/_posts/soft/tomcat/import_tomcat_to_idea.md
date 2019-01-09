---
layout: post
title: idea中编译tomcat源码(译)
date: 2015-12-28
categories: soft java_web
tags: tomcat 
---







## 创建子模块方式 

*   下载tomcat，这里用的7.0.42 [二进制发行版](http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.42/bin/apache-tomcat-7.0.42.tar.gz) 
*   下载tomcat [源码](http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.42/src/apache-tomcat-7.0.42-src.tar.gz)
*   创建工程,以下均以家目录作为相对路径进行操作 
    
        mkdir Tomcat
        tar -zxvf apache-tomcat-7.0.42.tar.gz
        tar -zxvf apache-tomcat-7.0.42-src.tar.gz
        mv apache-tomcat-7.0.42 catalina-home
        mv apache-tomcat-7.0.42-src tomcat-source
        touch pom.xml
        
*   编辑pom.xml

        <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
        
            <modelVersion>4.0.0</modelVersion>
            <groupId>net.imtiger</groupId>
            <artifactId>tomcat-study</artifactId>
            <name>Tomcat 7.0 Study</name>
            <version>1.0</version>
            <packaging>pom</packaging>
        
            <modules>
                <module>tomcat-source</module>
            </modules>
        </project>
    
*   进入tomcat-source目录，编辑pom.xml

        cd tomcat-source
        touch pom.xml

*   编辑子模块的pom.xml，下载的代码不符合maven默认的目录结构约定，需作修改更改Resources和TestResources,Sources和TestSources目录存放地

        <?xml version="1.0" encoding="UTF-8"?>
        <project xmlns="http://maven.apache.org/POM/4.0.0"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
        
        
            <modelVersion>4.0.0</modelVersion>
            <groupId>org.apache.tomcat</groupId>
            <artifactId>Tomcat7.0</artifactId>
            <name>Tomcat7.0</name>
            <version>7.0</version>
        
            <build>
                <finalName>Tomcat7.0</finalName>
                <sourceDirectory>java</sourceDirectory>
                <testSourceDirectory>test</testSourceDirectory>
                <resources>
                    <resource>
                        <directory>java</directory>
                    </resource>
                </resources>
                <testResources>
                    <testResource>
                        <directory>test</directory>
                    </testResource>
                </testResources>
                <plugins>
                    <plugin>
                        <groupId>org.apache.maven.plugins</groupId>
                        <artifactId>maven-compiler-plugin</artifactId>
                        <version>2.3</version>
        
                        <configuration>
                            <encoding>UTF-8</encoding>
                            <source>1.6</source>
                            <target>1.6</target>
                        </configuration>
                    </plugin>
                </plugins>
            </build>
        
            <dependencies>
                <dependency>
                    <groupId>junit</groupId>
                    <artifactId>junit</artifactId>
                    <version>4.4</version>
                    <scope>test</scope>
                </dependency>
                <dependency>
                    <groupId>ant</groupId>
                    <artifactId>ant</artifactId>
                    <version>1.7.0</version>
                </dependency>
                <dependency>
                    <groupId>wsdl4j</groupId>
                    <artifactId>wsdl4j</artifactId>
                    <version>1.6.2</version>
                </dependency>
                <dependency>
                    <groupId>javax.xml</groupId>
                    <artifactId>jaxrpc</artifactId>
                    <version>1.1</version>
                </dependency>
                <dependency>
                    <groupId>org.eclipse.jdt.core.compiler</groupId>
                    <artifactId>ecj</artifactId>
                    <version>4.2.2</version>
                </dependency>
            </dependencies>
        
        </project>

*   配置运行参数

        -Dcatalina.home=catalina-home -Dcatalina.base=catalina-home
        -Djava.endorsed.dirs=catalina-home/endorsed -Djava.io.tmpdir=catalina-home/temp
        -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager
        -Djava.util.logging.config.file=catalina-home/conf/logging.properties

*   走你！http://127.0.0.1:8080/

## 子模块法简易版 
    
   1.   下载tomcat某一发行版 以7.0.42 为例 [二进制发行版](http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.42/bin/apache-tomcat-7.0.42.tar.gz) 
   2.   下载对应发行版源码 [源码](http://archive.apache.org/dist/tomcat/tomcat-7/v7.0.42/src/apache-tomcat-7.0.42-src.tar.gz)
   3.   二进制减压到文件tomcat_xx目录下, 源码解压到tomcat目录下
   4.   复制tomcat_xx到tomcat中
   5.   进入tomcat目录,生成pom.xml,内容跟上边第二个pom.xml一样
   6.   打开你的idea,跑起来吧,入口是Bootstrap#main()
  
或者你可以直接fork我导入好的工程[tomcat_study](https://github.com/lcj1992/tomcat_study)

## eclipse 转 idea工程 

tomcat支持编译为eclipse工程，可以先编译为eclipse工程，然后用idea导入eclipse工程也可。详见参考[2]

## web项目目录结构 
webapp　　是工程的根路径

WEB-INF　打包后java的资源文件以及webapp原有的文件都会在这个目录下

*   web-app
    *   WEB-INF
        *   spring-mvc.xml
        *   db.xml
        *   classes
            *   Fxxk.class
        *   lib
            *   Fxxk.jar
    *   js
        *   jquery.js
    *   css


## 参考 

[1]<http://bbs.paris8.org/redirect.php?tid=8474&goto=lastpost>

[2]<http://tomcat.apache.org/tomcat-7.0-doc/building.html>
