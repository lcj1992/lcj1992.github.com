---
layout: post
title: dtd
date: 2015-12-27
tags:
    - dtd
---

如果tomcat web.xml中某个参数你忘记含义了，看dtd
如果你忘了某个参数名了，看dtd
...(不过现在都用xsd了)

    <web-app>
    <!DOCTYPE web-app PUBLIC
            "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
            "http://java.sun.com/dtd/web-app_2_3.dtd" >

    </web-app>

### 声明

##### 声明在xml文档中

     <?xml version="1.0"?>
    <!DOCTYPE note [
      <!ELEMENT note (to,from,heading,body)>
      <!ELEMENT to      (#PCDATA)>
      <!ELEMENT from    (#PCDATA)>
      <!ELEMENT heading (#PCDATA)>
      <!ELEMENT body    (#PCDATA)>
    ]>
    <note>
      <to>George</to>
      <from>John</from>
      <heading>Reminder</heading>
      <body>Don't forget the meeting!</body>
    </note>

#### 作为一个外部引用（<!DOCTYPE 根元素 SYSTEM "文件名"> ）
xml:

    <?xml version="1.0"?> <!DOCTYPE note SYSTEM "note.dtd">
    <note>
    <to>George</to>
    <from>John</from>
    <heading>Reminder</heading>
    <body>Don't forget the meeting!</body>
    </note>

note.dtd

    <!ELEMENT note (to,from,heading,body)>
    <!ELEMENT to (#PCDATA)>
    <!ELEMENT from (#PCDATA)>
    <!ELEMENT heading (#PCDATA)>
    <!ELEMENT body (#PCDATA)>

### 语法

#### 构建模块
所有的xml文档（以及html文档）均由以下简单的构建模块构成：

元素   XML 以及 HTML 文档的主要构建模块。元素可包含文本、其他元素或者是空的。如html的body，table等，上例中的note，to，from等，
属性   属性可提供有关元素的额外信息。属性总是被置于某元素的开始标签中。属性总是以名称/值的形式成对出现的。如\<img src="computer.gif" /\>
实体   实体是用来定义普通文本的变量。当文档被 XML 解析器解析时，实体就会被展开。如
PCDATA  被解析的字符数据（parsed character data）。不应当包含任何 &、< 或者 > 字符；需要使用 &、< 以及 > 实体来分别替换它们
CDATA  不被解析器解析的文本（character data）。在这些文本中的标签不会被当作标记来对待，其中的实体也不会被展开

#### 定义

##### 声明元素

    <!ELEMENT 元素名称 类别>
    <!ELEMENT 元素名称 (元素内容)>
    <!ELEMENT 元素名称 EMPTY>
    <!ELEMENT 元素名称 (#PCDATA)>
    <!ELEMENT 元素名称 ANY>
    <!ELEMENT 元素名称 (子元素名称 1,子元素名称 2,.....)>

    ...正则

##### 声明属性

    <!ATTLIST 元素名称 属性名称 属性类型 默认值>

##### 声明实体

    <!ENTITY 实体名称 "实体的值">
    xml中实体的使用：一个和号 (&), 一个实体名称, 以及一个分号 (;)。

### 作用

定义xml文档的合法构建模块

1.通过dtd，每个xml文件均可以携带一个有关其自身格式的描述。

2.可一致的使用某个标准的DTD来交换数据

3.通过dtd来验证从外部收到的数据

### 参考
[1]<http://www.w3school.com.cn/dtd/index.asp>