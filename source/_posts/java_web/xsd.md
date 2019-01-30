---
layout: post
title: xsd
date: 2015-12-27
tags:
    - xsd
---

a schema is an abstract collection of metadata, consisting of a set of schema components: chiefly element and attribute declarations and complex and simple type definitions. These components are usually created by processing a collection of schema documents, which contain the source language definitions of these components. In popular usage, however, a schema document is often referred to as a schema.

schema是元数据的抽象集合，包含一组schema 组件：元素和属性声明，复杂和简单的类型定义。 ...一个schema 文档通常被叫做schema。

#### 实用至上

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
           (所有使用spring bean 容器的，这一句是都是必须加的，几乎所有吧)
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          （这一句也是必须有的，引入xml schema instance命名空间，同时使用其schemaLocation 指出所使用的xsd的命名空间和具体位置）
           xmlns:context="http://www.springframework.org/schema/context"
          （这句和下两句一样都是看你自己，你的xml中除了基本命名空间还用到了哪些，就把其加进来）
           xmlns:aop="http://www.springframework.org/schema/aop"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.1.xsd
      http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd
      http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.1.xsd">
           （指出命名空间和其对应的xsd的位置）

#### xsd文件

    <?xml version="1.0"?>
    <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
    (表明 此schema 中用到的元素和数据类型来自命名空间 "http://www.w3.org/2001/XMLSchema"。
    同时它还规定了来自命名空间 "http://www.w3.org/2001/XMLSchema" 的元素和数据类型应该使用前缀 xs：)
    targetNamespace="http://www.w3school.com.cn"
    (表明 被此schema定义的元素(note, to, from, heading, body)来自命名空间： "http://www.w3school.com.cn"。)
    xmlns="http://www.w3school.com.cn"
    (表明 默认的命名空间是 "http://www.w3school.com.cn"。)
    elementFormDefault="qualified">
    (表明 任何XML实例文档所使用的且在此schema中声明过的元素必须被命名空间限定。)

    <xs:element name="note">
        <xs:complexType>
          <xs:sequence>
        <xs:element name="to" type="xs:string"/>
        <xs:element name="from" type="xs:string"/>
        <xs:element name="heading" type="xs:string"/>
        <xs:element name="body" type="xs:string"/>
          </xs:sequence>
        </xs:complexType>
    </xs:element>

    </xs:schema>

#### 在xml中使用xsd

    <?xml version="1.0"?>
    <note xmlns="http://www.w3school.com.cn"
    (规定了默认命名空间的声明。此声明会告知 schema 验证器，在此 XML 文档中使用的所有元素都被声明于 "http://www.w3school.com.cn" 这个命名空间.)
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    (xml schema实例 命名空间)
    xsi:schemaLocation="http://www.w3school.com.cn note.xsd">
    （xml schemaLocation 的schemaLocation属性，此属性有两个值，第一个值是需要使用的命名空间，第二个值是供命名空间使用的xml schema的位置）

    <to>George</to>
    <from>John</from>
    <heading>Reminder</heading>
    <body>Don't forget the meeting!</body>
    </note>

#### xsd语法
#### dtd继任者
基于xml编写

可扩展

支持数据类型

支持命名空间
