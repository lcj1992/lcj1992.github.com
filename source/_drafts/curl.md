---
layout: post
title: curl
date: 2015-12-27
categories: unix tool
tags:
    - curl
    - httpie
---

curl


提交数据 post请求 表单内容

    curl -d  "user=lcj&password=12345" http://www.google.com

返回header

    curl -i www.google.com

只返回header

    curl -I  www.google.com

使用代理

    curl -x  233.210.225.25:8085   http://www.google.com

保存cookie

    curl -D cookie0001.txt http://www.google.com

使用cookie -b 或--cookie

    curl -b cookie0001.txt http://www.google.com

使用user-agent

    curl -A  "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.0)"  -x  123.45.67.89:1080 http://www.google.com

伪造referer

    curl  -e "www.google.com"  http://mail.google.com

加强版的curl `httpie`
