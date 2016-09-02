---
layout: post
title: 解决maven冲突终极办法
categories: soft
tags: maven
---

#### 基础
1.打包命令：(爱起啥名起啥名，然后会按照目录结构生成jar，两个类都在bean目录下)

    jar cvf hehe.jar bean/UserCenterResponseBean.class  bean/UserInfoBean.class  

2.

*   groupId：组id，jar包所在的group，你会看到一些group名为com.google.guava等
*   artifactId：构件id，工程生成的构件，对应的就是com.google.guava的guava提供某一类功能，类库。
*   version：版本，每个artfact可能有多个版本，group下的artfactId+version，对应这一个明确的artfact（一般是jar包啦）

示例：

 <dependency>
     <groupId>org.mockito</groupId>
     <artifactId>mockito-core</artifactId>
     <version>${mockito-core.version}</version>
 </dependency>
对应的你会看到仓库中，命名是groupId + artfactId + version 路径下的artfactId+version.jar

![命名规则](/images/java/name.png)

#### 依赖

![依赖](/images/java/dependency.png)

#### 冲突
A -> B -> E1.0;

A -> C -> D -> E2.0

B -> X1.0

D ->X2.0

情景举例：包升级后，可能会出现一些类的变更，一般是增加类，如果你期望使用的高版本，但是实际使用的是低版本，就可能造成方法找不到，类找不到等之类的错误，出现NoSuchMethodError, ClassNotFoundException, can't create bean等错误。

1.包冲突：同一jar包

groupId和artifactId相同，version不同，没使用DependencyManagement，就会报包冲突，如果使用，则根据maven的管理策略，选择离项目最近的，但也可能出现上述错误。

2.类冲突：不同jar包

我打了个jar包，这个jar包中用了很多别人的类，包结构都没变就加进来了，又打了个jar，就可能出现这种情况；

两个jar包就是包含相同包结构的相同的类（这种情况应该也是存在的）

#### 定位解决
`mvn dependency:tree -Dverbose -Dincludes(-Dexcludes)=groupId:artifactId` 列出项目的依赖关系(可以指定只看某artfactId的) 检测类的依赖树

`mvn enforcer:enforce`规则校验，可以自定义规则，包冲突类冲突检测规则jar包的配置如下：

    <pluginManagement>
       <plugins>
           <plugin>
               <groupId>org.apache.maven.plugins</groupId>
               <artifactId>maven-enforcer-plugin</artifactId>
               <version>1.2</version>
               <dependencies>
                   <dependency>
                       <groupId>org.codehaus.mojo</groupId>
                       <artifactId>extra-enforcer-rules</artifactId>
                       <version>1.0-alpha-5</version>
                   </dependency>
               </dependencies>
           </plugin>
       </plugins>
    </pluginManagement>
    <plugins>
       <plugin>
           <groupId>org.apache.maven.plugins</groupId>
           <artifactId>maven-enforcer-plugin</artifactId>
           <configuration>
               <rules>
                   <banDuplicateClasses>
                       <findAllDuplicates>true</findAllDuplicates>
                       <ignoreClasses>
                           <ignoreClass>junit.*</ignoreClass>
                           <ignoreClass>org.junit.*</ignoreClass>
                           <ignoreClass>org.w3c.dom.*</ignoreClass>
                           <ignoreClass>javax.xml.namespace.*</ignoreClass>
                           <ignoreClass>org.apache.axis2.*</ignoreClass>
                       </ignoreClasses>
                       <message>[ERROR] [Trip Enforcer Rules] find DuplicateClasses</message>
                   </banDuplicateClasses>

                   <DependencyConvergence/>

               </rules>
           </configuration>
       </plugin>
    </plugins>

idea的show dependencies(ctrl + (shift) +alt + u)（好像用处不大。。//TODO）

#### 如何查看运行时出错的类到底load的是哪个jar包？

1.  根据错误信息，可以知道导致冲突的类名

2.  将下图代码放入程序，让程序跑起来，使用debug模式，设置断点，执行到断点，然后按alt+F8动态执行代码。

3.  按图所示，做就对了。

4.  在dependencyManagement中显式指定你想要的版本（nearest原理）,尽量不要使用exclude来解决包冲突依赖，太危险(运行时依赖是要走到这个路径才会用到的，一个庞大的系统，可能你现在没有触发这个路径，所以一直没有出问题。但是，如果一旦走到，那么就有问题了--昭辉)

5.  严重推荐使用dependencyManagement来管理依赖（下述）

出于对原作者的尊重：http://stamen.iteye.com/blog/2030552

![解决](/images/java/cope.png)

    import java.io.File;
    import java.net.MalformedURLException;
    import java.net.URL;
    import java.security.CodeSource;
    import java.security.ProtectionDomain;

    /**
    * 查看类所在的jar包
    **/
    public class ClassLocationUtils {
        public static String where(final Class cls) {
            if (cls == null) throw new IllegalArgumentException("null input: cls");
            URL result = null;
            final String clsAsResource = cls.getName().replace('.', '/').concat(".class");
            final ProtectionDomain pd = cls.getProtectionDomain();
            if (pd != null) {
                final CodeSource cs = pd.getCodeSource();
                if (cs != null) result = cs.getLocation();
                if (result != null) {
                    if ("file".equals(result.getProtocol())) {
                        try {
                            if (result.toExternalForm().endsWith(".jar") ||
                                    result.toExternalForm().endsWith(".zip"))
                                result = new URL("jar:".concat(result.toExternalForm())
                                        .concat("!/").concat(clsAsResource));
                            else if (new File(result.getFile()).isDirectory())
                                result = new URL(result, clsAsResource);
                        } catch (MalformedURLException ignore) {
                        }
                    }
                }
            }
            if (result == null) {
                final ClassLoader clsLoader = cls.getClassLoader();
                result = clsLoader != null ?
                        clsLoader.getResource(clsAsResource) :
                        ClassLoader.getSystemResource(clsAsResource);
            }
            return result.toString();
        }

    }

#### 日志框架
很多时候日志各种jar会冲突，单列出来说下。

![日志jar关系](/images/soft/log_relation.png)

|jar|类型|说明|
|-|-|-|
|org.slf4j:slf4j-api|接口，桥接|slf4j api,具体实现交给其他日志|
|org.apache.commons:commons-logging-api|接口，桥接|jcl api,同上，我们的应用中肯定不会以它为api，但是很多框架中使用了它|
|org.slf4j:jcl-over-slf4j|过渡包|使用slf4j接管jcl|
|org.slf4j:slf4j-jcl|过渡包|使用jcl接管slf4j|
|org.slf4j:log4j-over-slf4j|过渡包|使用slf4j接管log4j1，对于log4j2是不适用的|
|org.slf4j:slf4j-log4j12|过渡包|使用log4j接管slf4j|
|org.slf4j:jul-to-slf4j|过渡包|使用slf4j接管jul(java.util.logging)|
|org.apache.logging.log4j:log4j-jul|过渡包|把jul的接口适配到log4j2的实现|
|org.apache.logging.log4j:log4j-1.2-api|过渡包|把log4j1的接口适配到log4j2的实现|
|ch.qos.logback:logback-classic|日志实现||
|ch.qos.logback:logback-core|日志实现||
|org.apache.logging.log4j:log4j-api|日志实现||
|org.apache.logging.log4j:log4j-core|日志实现||
|org.apache.logging.log4j:log4j-slf4j-impl|过渡包|log4j2是可以不依赖任何桥接包slf4j，jcl，这个包是使用slf4j来接管log4j的。|

推荐实践为：

1.  slf4j + logback
2.  slf4j + log4j2

#### 接口统一使用slf4j接管

    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>${org.slf4j.version}</version>
    </dependency>

    <!--Jakarta Commons Logging redirect to slf4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jcl-over-slf4j</artifactId>
        <version>${org.slf4j.version}</version>
        <scope>runtime</scope>
    </dependency>

    <!--Apache log4j redirect to slf4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>log4j-over-slf4j</artifactId>
        <version>${org.slf4j.version}</version>
        <scope>runtime</scope>
    </dependency>

    <!--Java Util Logging redirect to slf4j -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>jul-to-slf4j</artifactId>
        <version>${org.slf4j.version}</version>
        <scope>runtime</scope>
    </dependency>

##### 实现使用logback

    <!--slf4j的配置-->

    <!--logback的配置-->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>${logback.version}</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-core</artifactId>
        <version>${logback.version}</version>
        <scope>runtime</scope>
    </dependency>

##### 实现使用log4j2

    <!--slf4j的配置-->

    <!--log4j2的配置-->
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-slf4j-impl</artifactId>
        <version>2.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.3</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.3</version>
    </dependency>

#### 常见冲突解决    

1. junit :jmockit 冲突问题: 请将 junit 换成 junit-dep

2. commons-beanutils & commons-collections 冲突问题: 删除commons-beanutils

3. jcl-over-slf4j & commons-logging 冲突: 删掉 commons-logging,类似日志的实践参考上方。

4. io.netty:netty & org.jboss.netty:netty 删掉后者（后者改名成前者了）

5. cglib-nodep & cglib cglib-nodep里包含了asm包，cglib里不包含asm包。asm包和cglib不匹配也会出错。因此用cglib-nodep就不会出现版本不匹配情况

6. fastxml  codehaus  后者已经不维护了，改名为fastxml了

#### 参考

[slf4j]<http://www.slf4j.org/manual.html>

[日志框架]<http://www.cnblogs.com/enjiex/p/3732338.html>

[日志框架]<http://wiki.sankuai.com/pages/viewpage.action?pageId=575620751>
