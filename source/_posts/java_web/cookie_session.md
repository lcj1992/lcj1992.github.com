---
layout: post
title: cookie和session
date: 2016-01-07
categories: java_web
tags:
    - cookie
    - session
---

### cookie

HTTP cookie ***由`服务端（产生方）`发送到`客户端（存储方）`，并存储在用户的客户端（浏览器）中．每次用户请求网站时，浏览器会将cookie发回给服务器．***cookie可以用来记录一些状态信息（如购物车中的商品）或者记录用户的浏览行为

#### 属性

|属性|说明|
|-|-|
|name|cookie名|
|value|cookie值|
|domain|哪个域名发送的，最长256字符|
|path|哪个路径发送的|
|secure|是否允许|
|httponly|是否仅能通过http和https渠道暴漏．防止客户端js执行，盗取cookie|
|max-age|持续时间|
|expires|过期时间,和上类似|

[cookie的实现](#java_cookie)

servlet-api 2.5之前版本（含）没有httpOnly这个字段，需要费点劲来设置httpOnly `cookie.setPath(isHttpOnly ? "/pathxx; HttpOnly;": "/pathxx");`

![cookie](/images/web/cookie.png)

ps:

1.  对于一些静态资源的访问,如css,script等,发送cookie是没有意义的,所以一般静态资源会使用独立的域名,避免请求静态资源时发送cookie,减少cookie传输的次数.
例如去哪儿的静态资源使用的域名是qunarzz.com而不是qunar.com

2.  chrome中的cookie,***如果cookie不设置过期时间(`会话cookie`,就是maxAge=-1),表示这个cookie的生命周期为浏览器会话时期***，下图的过期时间那列Session就是这个意思

3.  没有了cookie，网站似乎都玩不转。亲测淘宝、qq邮箱、百度、去哪儿的网站。

比如禁止了cookie，在你登录时，百度会直接提示你

![禁止了cookie百度直接提示你](/images/java_web/no_cookie_baidu.png)

#### cookie实践

结合我的工作经历，我见过的cookie的使用方法：

1.  追踪用户行为，做trace使，在用户首次进入网站时，种下cookie。之后的请求在interceptor中拦截获取cookie打到日志mdc中。

2.  做缓存的key,避免不必要的查库操作。比如退票时，分为退票申请、计算手续费、退票确认三步，但是用户的三次请求中订单信息等是不变的，可以只在退票申请时查db，然后写入缓存、并种下cookie；之后的计算手续费、退票确认都不需查db，直接从缓存中取即可。

3. 登录 todo

### session

session需要使用cookie作为识别标志。HTTP协议是无状态的，session不能依据HTTP连接来判断是否为同一客户，因此服务器向客户端浏览器发送一个名为JSESSIONID的cookie，其值为该session的ID。该cookie为服务器自动生成的，***maxAge属性一般为-1，表示仅当前浏览器内有效，并且各浏览器窗口间不共享，关闭浏览器就会失效。***因此同一机器的两个浏览器窗口访问服务器时会生成两个不同的session

##### cookie的java实现

    //
    // The value of the cookie itself.
    //

    private String name;	// NAME= ... "$Name" style is reserved
    private String value;	// value of NAME

    //
    // Attributes encoded in the header's cookie fields.
    //

    private String comment;	// ;Comment=VALUE ... describes cookie's use
				// ;Discard ... implied by maxAge < 0
    private String domain;	// ;Domain=VALUE ... domain that sees cookie
    private int maxAge = -1;	// ;Max-Age=VALUE ... cookies auto-expire
    private String path;	// ;Path=VALUE ... URLs that see the cookie
    private boolean secure;	// ;Secure ... e.g. use SSL
    private int version = 0;	// ;Version=1 ... means RFC 2109++ style
    private boolean isHttpOnly = false;


参考：

[1]<https://en.wikipedia.org/wiki/HTTP_cookie>

[2]<http://www.cnblogs.com/shiyangxt/archive/2008/10/07/1305506.html>
