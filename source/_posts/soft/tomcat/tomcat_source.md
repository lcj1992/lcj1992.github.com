---
layout: post
title: tomcat的启动关闭与请求处理
date: 2016-04-26
categories: soft
tags:
    - tomcat 
---

基于7.0.42.0, tomcat源码导入idea[参看这篇](/2015/12/28/import_tomcat_to_idea)

#### 整体架构

![tomcat架构](/images/soft/tomcat_structure.jpg)

1.  `Server`(服务器)顶级构成元素,所有一切包含在Server中,Server的实现类StandardServer可以包含一个到多个service
2.  `Service`(服务)次顶级元素,其实现类为StandardService调用Container(容器)的接口,实际是调用Engine,而且StandardService类中也指明了该Service归属的Server
3.  `Container` (容器) Host(主机),Context(上下文),Engine(引擎)均继承自Container接口.但是他们是有父子关系的,在Host(主机),Context(上下文)和Engine(引擎)这三类容器中,Engine是顶级容器,直接包含Host,而Host又包含Context,所以Engine,Host和Context从大小上来说又构成父子关系.
4.  `Connector` (连接器) 将Service和Container链接起来,首先它需要注册到一个Service,它的作用就是把来自客户端的请求转达到Container,这就是它为什么称作连接器的原因.

所以从功能的角度将tomcat分为5个子模块,他们分别是:

1.  Jasper模块: 负责jsp页面的解析,jsp属性的验证,同时也负责将jsp页面动态转换为java代码并编译成class文件,tomcat源码中org.apache.jasper包及其自爆的源码都属于这个子模块
2.  Servlet和Jsp规范的实现模块: 源码位于javax.servlet包及其子包,javax.servlet.Servlet接口,javax.servlet.http.httpServlet类等就位于这个子模块
3.  Catalina子模块: 包含org.apache.catalina开头的源码.该子模块的任务是规范Tomcat的总体架构,定义了Server,Service,Host,Connector,Context,Session及Cluster等关键组件及这些组件的实现,这个子模块大量运用了Composite设计模式.同时也规范了Catalina的启动关闭等事件的执行流程.
4.  Connectors子模块:  如果说上面三个子模块实现了tomcat应用服务器的话,那么这个子模块就是web服务器的实现.所谓连接器就是一个连接客户和应用服务器的桥梁,他接收用户的请求,并把用户请求包装成标准的Http请求(包含协议名称,请求头Head,请求方Get或者Post等)
同时在请求页面未发现时,connector就会给客户端浏览器发送标准的Http 404错误相应页面
5.  Resource子模块: 这个子模块包含一些资源文件,如Server.xml及Web.xml配置文件.严格来说,这个子模块不包含java源代码,但是它是tomcat编译运行所必需的.

#### 启动关闭流程

这个日志熟悉么? 不熟悉的话,重启下你机器上的任一tomcat项目,熟悉一下

    May 07, 2016 2:13:51 PM org.apache.catalina.core.StandardServer await
    INFO: A valid shutdown command was received via the shutdown port. Stopping the Server instance.
    May 07, 2016 2:13:51 PM org.apache.coyote.AbstractProtocol pause
    INFO: Pausing ProtocolHandler ["http-bio-8080"]
    May 07, 2016 2:13:51 PM org.apache.catalina.core.StandardService stopInternal
    INFO: Stopping service Catalina
    May 07, 2016 2:13:52 PM org.apache.catalina.loader.WebappClassLoader clearReferencesThreads
    SEVERE: The web application [] appears to have started a thread named [AsyncAppender-Worker-Thread-3] but has failed to stop it. This is very likely to create a memory leak.
    ay 07, 2016 2:13:53 PM org.apache.coyote.AbstractProtocol stop
    INFO: Stopping ProtocolHandler ["http-bio-8080"]
    May 07, 2016 2:13:53 PM org.apache.coyote.AbstractProtocol destroy
    INFO: Destroying ProtocolHandler ["http-bio-8080"]
    May 07, 2016 2:13:55 PM org.apache.catalina.loader.WebappClassLoader loadClass
    INFO: Illegal access: this web application instance has been stopped already.  Could not load org.I0Itec.zkclient.ZkClient$8.  The eventual following stack trace is caused by an error thrown for debugging purposes as well as to attempt to
    ...

    May 06, 2016 10:59:56 PM org.apache.coyote.AbstractProtocol init
    INFO: Initializing ProtocolHandler ["http-bio-8080"]
    May 06, 2016 10:59:56 PM org.apache.catalina.startup.Catalina load
    INFO: Initialization processed in 1157 ms
    May 06, 2016 10:59:56 PM org.apache.catalina.core.StandardService startInternal
    INFO: Starting service Catalina
    May 06, 2016 10:59:56 PM org.apache.catalina.core.StandardEngine startInternal
    INFO: Starting Servlet Engine: Apache Tomcat/7.0.47
    May 06, 2016 10:59:56 PM org.apache.catalina.startup.HostConfig deployDirectory
    INFO: Deploying web application directory /xxx.com/webapps/ROOT
    May 06, 2016 10:59:58 PM org.apache.coyote.AbstractProtocol start
    INFO: Starting ProtocolHandler ["http-bio-8080"]
    May 06, 2016 10:59:58 PM org.apache.catalina.startup.Catalina start
    INFO: Server startup in 1818 ms

tomcat的入口为`BootStrap#main`

    Bootstrap loader for Catalina.  This application constructs a class loader
    for use in loading the Catalina internal classes (by accumulating all of the
    JAR files found in the "server" directory under "catalina.home"), and
    starts the regular execution of the container.  The purpose of this
    roundabout approach is to keep the Catalina internal classes (and any
    other classes they depend on, such as an XML parser) out of the system
    class path and therefore not visible to application level classes.

大致意思:Catalina的启动加载器,加载catalina.home目录下的classes,并启动容器.

这样保证了Catalina内部的类不在系统的path下,因此对于其他应用是不可见的(应用隔离)(自定义类加载器的用途)

启动:

1.  Bootstrap#init

    *   1.1 设置catalina.home,设置catalina.base (ps: [CATALINA_HOME vs CATALINA_BASE](https://tomcat.apache.org/tomcat-8.0-doc/introduction.html#Directories_and_Files))
    *   1.2 初始化类加载器,`commonLoader`,`catalinaLoader`,`sharedLoader`
    *   1.3 用catalinaLoader对`org.apache.catalina.startup.Catalina`类进行加载,并进行实例化startupInstance,设置catalinaDaemon为startupInstance
    *   1.4 在bootstrap初始化完毕之后,设置daemon为bootstrap

2.  根据传入命令, 已start为例,加载,调用Bootstrap#load(args),实际上是调用catalinaDaemon#load方法

    *   2.1 Catalina#createStartDigester 创建解析规则, 就是创建Server,Service,Executor等元素与相关类的对应关系,根据其配置实例化相应模块,参见[server.xml](https://github.com/lcj1992/tomcat_study/blob/master/conf/server.xml),

            digester.addObjectCreate("Server","org.apache.catalina.core.StandardServer","className");
            digester.addSetProperties("Server");
            digester.addSetNext("Server","setServer","org.apache.catalina.Server");

            digester.addObjectCreate("Server/Listener",null, // MUST be specified in the element(必须在你的server.xml中制定)
                                             "className");
            digester.addSetProperties("Server/Listener");
            digester.addSetNext("Server/Listener","addLifecycleListener","org.apache.catalina.LifecycleListener");

    *   2.2 读取`$CATALINA_BASE/conf/server.xml`,然后解析之,digester#parse(inputSource),会实例化一个Server,并Catalina#setServer指向这个实例,并将Server#setCatalina指向这个Catalina实例.
    开始调用StandardServer#initInternal,初始化server实例,初始化时会对状态机进行校验  
        *   globalNamingResources的初始化  
        *   依据我们server.xml中的我们的配置,初始化services(可多个)(StandardService)  
            *   初始化Engine(StandardEngine),创建一个线程池,用于启动和停止服务  
            *   初始化Executors(可多个)(StandardThreadExecutor)  
            *   初始化Connectors(可多个)(Connector)  
                *   初始化protocolHandler(eg: Http11Protocol)    
                    *   初始化endpoint(eg JioEndpoint),然后endpoint#bind(),绑定地址和端口,设置线程池的大小,并创建`serverSocket` EndPoint的bindState 有UNBOUND -> BOUND_ON_START  
                *   初始化mapperListener
    *   2.3 启动各组件,还是调用Catalina#start(),启动时都会动状态机进行校验
        *   globalNamingResources的启动
        *   启动各services
            *   启动Engine
                *   findChildren() 获取子容器,比如StandardHost的实例,将这些子容器用StartChild类包装成Callable的类，使用线程池启动
                    *   child#start() -> (eg)StandardHost#startInternal() -> setState(LifecycleState.STARTING) -> LifecycleBase#setState -> LifecycleBase#setStateInternal -> LifecycleBase#fireLifecycleEvent(LifecycleBase的实现为EngineConfig)
                        *   HostConfig#deployApps(deploy configBase的xml文件,war包s,加压后的项目s等)
                            *   以DeployWars为例,new一个DeployWar(HostConfig config,ContextName cn,File war),DeployWar实际上是一个runnable,然后将其扔进host.getStartStopExecutor()中
            *   启动Executors
            *   启动connectors
                *   启动protocol
                    *   启动endpoint,初始化connectionLimitLatch,启动`acce  ptorThreads`,启动timeoutThread
                *   启动mapperListener, findDefaultHost(),addListeners(engine),registerHost(host)
    *   2.4 如果useShutdownHook为true,添加CatalinaShutdownHook`
    *   2.5 Catalina#await(),new 一个server socket to wait on (默认端口号为8005,你懂的)

#### 状态机  

org.apache.Catalina.Lifecycle的实现类都具有如下的[状态机](https://github.com/lcj1992/tomcat_study/blob/master/java/org/apache/catalina/Lifecycle.java).

                 start()
       -----------------------------
       |                           |
       | init()                    |
      NEW ->-- INITIALIZING        |
      | |           |              |     ------------------<-----------------------
      | |           |auto          |     |                                        |
      | |          \|/    start() \|/   \|/     auto          auto         stop() |
      | |      INITIALIZED -->-- STARTING_PREP -->- STARTING -->- STARTED -->---  |
      | |         |                                                  |         |  |
      | |         |                                                  |         |  |
      | |         |                                                  |         |  |
      | |destroy()|                                                  |         |  |
      | -->-----<--       auto                    auto               |         |  |
      |     |       ---------<----- MUST_STOP ---------------------<--         |  |
      |     |       |                                                          |  |
      |    \|/      ---------------------------<--------------------------------  ^
      |     |       |                                                             |
      |     |      \|/            auto                 auto              start()  |
      |     |  STOPPING_PREP ------>----- STOPPING ------>----- STOPPED ---->------
      |     |                                ^                  |  |  ^
      |     |               stop()           |                  |  |  |
      |     |       --------------------------                  |  |  |
      |     |       |                                  auto     |  |  |
      |     |       |                  MUST_DESTROY------<-------  |  |
      |     |       |                    |                         |  |
      |     |       |                    |auto                     |  |
      |     |       |    destroy()      \|/              destroy() |  |
      |     |    FAILED ---->------ DESTROYING ---<-----------------  |
      |     |                        ^     |                          |
      |     |     destroy()          |     |auto                      |
      |     -------->-----------------    \|/                         |
      |                                 DESTROYED                     |
      |                                                               |
      |                            stop()                             |
      --->------------------------------>------------------------------

1.  任何状态都可以过渡到FAILED
2.  当组件在STARTING_PREP,STARTING,STARTED,调用start()方法是没有作用的
3.  当组件在NEW状态时调用start()方式会先调用init()
4.  当组件在STOPPING_PREP,STOPPING,STOPPED调用stop()方法是不起作用的
5.  当组件从NEW过渡到STOPPED会调用stop()方法.这经常发生在一个组件没有启动起来,并且没有启动他的所有子组件.当一个组件stopped后,它会试着停止所有它的子组件,即使它没有启动起来
6.  MUST_STOP用来标明从start()中退出后,应该调用stop().这经常用在一个组件启动失败了.
7.  MUST_DESTROY用来标明从stop()中退出后,应该调用destroy(),这经常用在不需要重启的组件.
8.  任何别的状态过渡都会抛出LifecycleException
9.  在调用其中方法时触发状态变更,触发LifecycleEvents

Lifecycle的子类类图:

![Lifecycle子类](/images/soft/tomcat_lifecycle.png)

#### tomcat处理请求

![tomcat处理请求](/images/soft/tomcat_handle_request.jpg)

对于BIO模式,tomcat启动后就是使用JioEndPoint.Acceptor来接收外部请求的socket连接. 启动个tomcat,jstack下,你会发现如下线程栈.

    "http-bio-8080-Acceptor-0" daemon prio=5 tid=0x00007fb583ae0000 nid=0x5503 runnable [0x0000700001555000]
    java.lang.Thread.State: RUNNABLE
	at java.net.PlainSocketImpl.socketAccept(Native Method)
	at java.net.AbstractPlainSocketImpl.accept(AbstractPlainSocketImpl.java:398)
	at java.net.ServerSocket.implAccept(ServerSocket.java:530)
	at java.net.ServerSocket.accept(ServerSocket.java:498)
	at org.apache.tomcat.util.net.DefaultServerSocketFactory.acceptSocket(DefaultServerSocketFactory.java:60)
	at org.apache.tomcat.util.net.JIoEndpoint$Acceptor.run(JIoEndpoint.java:216)
	at java.lang.Thread.run(Thread.java:745)

JioEndpoint的Acceptor是这么产生的,启动`getAcceptorThreadCount()`个线程,接收请求.

>   Connector#startInternal() -> AbstractEndpoint#start() -> JIoEndpoint#startInternal() -> AbstractEndpoint#startAcceptorThreads() -> JioEndpoint#createAcceptor()

![create_acceptor](/images/soft/create_acceptor.png)

其中AbstractEndpoint#start():

    public final void start() throws Exception {
        if (bindState == BindState.UNBOUND) {
            bind();
            bindState = BindState.BOUND_ON_START;
        }
        startInternal();
    }

    protected final void startAcceptorThreads() {
        int count = getAcceptorThreadCount();
        acceptors = new Acceptor[count];

        for (int i = 0; i < count; i++) {
            acceptors[i] = createAcceptor();
            String threadName = getName() + "-Acceptor-" + i;
            acceptors[i].setThreadName(threadName);
            Thread t = new Thread(acceptors[i], threadName);
            t.setPriority(getAcceptorThreadPriority());
            t.setDaemon(getDaemon());
            t.start();
        }
    }

接下来就是接收socket,处理socket

1.  countUpAwaitConnection(),如果maxConnections = -1直接返回,否则,acquire 一个共享的latch或者等会
2.  socket = socket.accept(serverSocket)
3.  processSocket(socket),对于Bio http源码如下:

        protected boolean processSocket(Socket socket) {
        try {
            SocketWrapper<Socket> wrapper = new SocketWrapper<Socket>(socket);
            wrapper.setKeepAliveLeft(getMaxKeepAliveRequests());
            // During shutdown, executor may be null - avoid NPE
            if (!running) {
                return false;
            }
            getExecutor().execute(new SocketProcessor(wrapper));
        } catch (RejectedExecutionException x) {
            log.warn("Socket processing request was rejected for:"+socket,x);
            return false;
        } catch (Throwable t) {
            ExceptionUtils.handleThrowable(t);
            // This means we got an OOM or similar creating a thread, or that
            // the pool and its queue are full
            log.error(sm.getString("endpoint.process.fail"), t);
            return false;
        }
        return true;

然后返回请求, 请求返回的方法调用链:

1.  JIoEndpoint$SocketProcessor#run  
2.  AbstractProtocol$AbstractConnectionHandler#process  
3.  AbstractHttp11Processor#process  
4.  CoyoteAdapter#service
5.  StandardEngineValve#invoke  
6.  AccessLogValve#invoke
7.  ErrorReportValve#invoke  
8.  StandardHostValve#invoke
9.  AuthenticatorBase#invoke  
10. StandardContextValve#invoke  
11. StandardWrapperValve#invoke  
12. ApplicationFilterChain#doFilter  
13. ApplicationFilterChain#internalDoFilter
14. HttpServlet#service 然后调用doGet 或者doPost

[HTTPServlet源码](https://github.com/lcj1992/tomcat_study/blob/master/java/javax/servlet/http/HttpServlet.java)

这段日志熟悉不.不熟悉的话,自己写个controller,主动throw个异常

    at com.xx.controller.BookController.order(BookController.java:114) [BookController.class:na]
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.7.0_45]
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57) ~[na:1.7.0_45]
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43) ~[na:1.7.0_45]
    at java.lang.reflect.Method.invoke(Method.java:606) ~[na:1.7.0_45]
    at org.springframework.web.method.support.InvocableHandlerMethod.invoke(InvocableHandlerMethod.java:219) [spring-web-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:132) [spring-web-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:100) [spring-webmvc-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:604) [spring-webmvc-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:565) [spring-webmvc-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:80) [spring-webmvc-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:923) [spring-webmvc-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:852) [spring-webmvc-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:882) [spring-webmvc-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.servlet.FrameworkServlet.doGet(FrameworkServlet.java:778) [spring-webmvc-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:621) [servlet-api.jar:na]
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:728) [servlet-api.jar:na]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:305) [catalina.jar:7.0.47]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:210) [catalina.jar:7.0.47]
    at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:51) [tomcat7-websocket.jar:7.0.47]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:243) [catalina.jar:7.0.47]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:210) [catalina.jar:7.0.47]
    at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:88) [spring-web-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:76) [spring-web-3.1.4.RELEASE.jar:3.1.4.RELEASE]
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:243) [catalina.jar:7.0.47]
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:210) [catalina.jar:7.0.47]
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:222) [catalina.jar:7.0.47]
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:123) [catalina.jar:7.0.47]
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:502) [catalina.jar:7.0.47]
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:171) [catalina.jar:7.0.47]
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:100) [catalina.jar:7.0.47]
    at org.apache.catalina.valves.AccessLogValve.invoke(AccessLogValve.java:953) [catalina.jar:7.0.47]
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:118) [catalina.jar:7.0.47]
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:408) [catalina.jar:7.0.47]
    at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1041) [tomcat-coyote.jar:7.0.47]
    at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:603) [tomcat-coyote.jar:7.0.47]
    at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.java:310) [tomcat-coyote.jar:7.0.47]
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145) [na:1.7.0_45]
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615) [na:1.7.0_45]
    at java.lang.Thread.run(Thread.java:744) [na:1.7.0_45]

#### 参考

[tomcat8官方文档]<https://tomcat.apache.org/tomcat-8.0-doc/config/service.html>

[tomcat的启动和关闭]<http://yikebocai.com/2014/11/tomcat-source-code-study-2/>

[tomcat处理请求]<http://yikebocai.com/2014/11/tomcat-source-code-study-3/>

[tomcat处理HTTP请求源码分析]<http://www.infoq.com/cn/articles/zh-tomcat-http-request-1>
