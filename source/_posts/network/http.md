---
layout: post
title: http协议(译)
date: 2015-12-27
categories: network
tags:
    - http
---

`Each header field consists of a name followed by a colon (":") and the field value. Field names are case-insensitive.`

每个header域包含一个名字，冒号和值，域名是大小写敏感的．域值从冒号后第一个非空字符开始，头域可以被扩展为多行，在每行开始处，使用至少一个空格或制表符。

#### 请求报文头

|header field name|描述|例子|
|-|-|-|
|Accept|响应可以接受的内容类型[详见](https://en.wikipedia.org/wiki/Content_negotiation)|`Accept:text/plain`|
|Accept-Charset|可以接受的字符集|`Accept-Charset:utf-8`|
|Accept-Encoding|可以接受的编码类型[详见](https://en.wikipedia.org/wiki/Content_negotiation)|`Accept-Encoding: gzip, deflate`|
|Accept-Language|可以接受的返回的语言(汉语，英语..)[详见](https://en.wikipedia.org/wiki/Content_negotiation)|`Accept-Language: en-US`|
|[Cache-Control](https://en.wikipedia.org/wiki/Web_cache#Cache_control)|请求响应必须遵守的缓存机制|`Cache-Control: no-cache`|
|Connection|当前连接的控制选项|`Connection:keep-alive`      `Connection:Upgrade`|
|Cookie|[cookie](https://en.wikipedia.org/wiki/HTTP_cookie)是由服务器之前就设置好的[Set-Cookie](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#innerlink_set-cookie)|`Cookie: $Version=1; Skin=new;`|
|Content-length|请求体的长度(单位是字节)|`Content-Length: 348`|
|Content-MD5|请求体通过[Base64](https://en.wikipedia.org/wiki/Base64)的MD5加密的结果|`Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==`|
|Content-Type|请求体的媒体类型[MIME type](https://en.wikipedia.org/wiki/Media_type)(用在POST和PUT方法上)|`Content-Type: application/x-www-form-urlencoded`|
|Date|消息被发送的时间|`Date: Tue, 15 Nov 1994 08:12:31 GMT`|
|Host|服务器的域名和服务监听的端口号(可以省略)|`Host: en.wikipedia.org:80`      `Host: en.wikipedia.org`|
|Origin|Initiates a request for [cross-origin resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (asks server for an 'Access-Control-Allow-Origin' response field) .|`Origin: http://www.example-social-network.com`|
|Referer|允许的能跳转到当前页面的页面|`Referer: http://en.wikipedia.org/wiki/Main_Page` |
|User-Agent|用户代理|`User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:12.0) Gecko/20100101 Firefox/21.0`|

#### 响应报文头

|field name|描述|例子|
|-|-|-|
|Access-Control-Allow-Origin|表明哪些web可以参与[cross-origin-resource sharing](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing)|`Access-Control-Allow-Origin: *` |
|Age|已经cache的时间seconds|`Age: 12`|
|Allow|允许的方法|`Allow: GET, HEAD`|
|Cache-Control|缓存机制|`Cache-Control: max-age=3600`|
|Connection|当前连接的控制选项|`Connection: close`|
|Content-Encoding|编码类型|`Content-Encoding: gzip`|
|Content-Language|语言(汉语，英语..)|`Content-Language: da`|
|Content-Length|response的长度(字节数)|`Content-Length: 348`|
|Content-Location|一个可选的位置|`Content-Location: /index.htm`|
|Content-MD5|请求体通过[Base64](https://en.wikipedia.org/wiki/Base64)的MD5加密的结果|`Content-MD5: Q2hlY2sgSW50ZWdyaXR5IQ==`|
|Content-Type|请求体的媒体类型[MIME type](https://en.wikipedia.org/wiki/Media_type)(用在POST和PUT方法上)|`Content-Type: text/html; charset=utf-8`|
|Date|时间|`Date: Tue, 15 Nov 1994 08:12:31 GMT`|
|ETag|||
|Last-Modified|被请求的对象上次修改的时间|`Last-Modified: Tue, 15 Nov 1994 12:45:26 GMT`|
|Location|用于[重定向](https://en.wikipedia.org/wiki/URL_redirection)|`Location: http://www.w3.org/pub/WWW/People.html`|
|Server|服务器名|`Server: Apache/2.4.1 (Unix)`|
|Status|状态码|`Status: 200 OK`|

[1]<http://en.wikipedia.org/wiki/List_of_HTTP_header_fields>

[2]<http://lvwenwen.iteye.com/blog/1570468>

#### 状态码

|状态码|常见状态码|含义|
|-|-|
|1xx| |`informational`|
|2xx| |`success`|
| |200 | OK |
|3xx| |`redirection`|
| |301 | Moved Permanently|
| |302 | Found |
|4xx| |`client error`|
| |400 | Bad Request |
| |401 | Unauthorized |
| |403 | Forbidden|
| |404 | Not Found|
| |405 | Method Not Allowed|
| |415 | Unsupported Media Type|
|5xx| |`server error`|
| |500| Internal Server Error|
| |502| Bad GateWay|

[1]<https://httpstatuses.com/>

[2]<http://en.wikipedia.org/wiki/List_of_HTTP_status_codes>
