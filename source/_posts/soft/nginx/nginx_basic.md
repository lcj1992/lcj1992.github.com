---
layout: post
title: nginx简单使用
date: 2015-12-28
categories: soft
tags: nginx
---




### basic

一个人访问http://foolchild.cn/，浏览器默认是80（listen）端口，然后会根据
location的映射关系访问 http://fuck/，再用127.0.0.1:8080或者10.86.41.3：8080替换其中的fuck
（server_name）+ （listen）+（location）是用户访问的网址，
经（upstream）处理后的（proxy_pass）是真实的服务器上的地址。

`server_name + listen + location` -> `经upstream替换后的proxy_pass`

     http {
         upstream fuck { 
             server 127.0.0.1:8080; 
             server 10.86.41.3:8080 
         }
        
         server {
             listen       80;
             server_name  foolchild.cn;
        
             location / {                 
                 proxy_pass  http://fuck/;             
             }
         }  
     }

set $app_name $1; $1为第一个/后的值,$2为第二个/后的值

### lua

公司的源里的nginx自带lua模块,直接安装就可以,但是可能会存在版本问题,

1.  更新软件源
2.  查看nginx依赖的动态链接库 ldd \`which nginx\`

http://pureage.info?strider=1&strider=2&strider=3&strider=4

1.local strider = ngx.var.arg_strider


只会去strider中的第一个   1

2.local strider = ngx.req.get_uri_args["strider"]

这个会取所有  1，2，3，4

参考

[1]<http://nginx.org/en/docs/>

[2]<http://www.cnblogs.com/xiaogangqq123/archive/2011/03/02/1969006.html>
