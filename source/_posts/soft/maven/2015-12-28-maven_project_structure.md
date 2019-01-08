---
layout: post
title: maven工程结构(译)
categories: soft
tags: maven
---

#### project
Whatever you do in IntelliJ IDEA, you do that in the context of a project. A project is an organizational unit that represents a complete software solution.

It serves as a basis for coding assistance, bulk refactoring, coding style consistency, etc.Your finished product may be decomposed into a series of discrete,

isolated modules, but it's a project definition that brings them together and ties them into a greater whole.

Projects don't themselves contain development artifacts such as source code, build scripts, or documentation.

They are the highest level of organization in the IDE, and they define project-wide settings as well as collections of what IntelliJ IDEA refers to as modules and libraries.

中文：

在idea中，无论你做什么，你都是在一个工程的上下文中。一个工程是代表了一个完整的软件solution的有组织的单元。

他作为代码协助，散装重构，代码风格一致性的基础。你最后完成的产品可能被分解成一系列离散的独立的模块，

但是它可以将他们聚合到一块成一个完整的工程。工程自身不包含要部署的artfacts例如源码，编译脚本或者文档。

他们是编译器中最高的组织层级，他们定义了一个工程性的设置，是一个模块和类库的结合。

#### module
A module is a discrete unit of functionality which you can compile, run, test and debug independently.

Modules contain everything that is required for their specific tasks: source code, build scripts, unit tests, deployment descriptors, and documentation.

However, modules exist and are functional only in the context of a project

Configuration information for a module is stored in a .iml module file. By default, such a file is located in the module's content root folder.

Development teams, normally, share the .iml module files through version control.

中文：

一个模块是一个可以独立编译，运行，测试，调试的功能性的离散单元。

模块包含特定任务的所有东西：源码，编译脚本，单元测试，部署描述，文档

然而，模块只能存在于工程中，只有在工程中才能起作用。

一个模块的配置信息被存储于.iml文件中。默认情况这个文件放在模块的根目录下。

开发团队，正常情况下，通过版本控制共享.iml模块文件。

#### facets
Facets represent various frameworks, technologies and languages used in a module.

They let IntelliJ IDEA know how to treat the module contents and thus ensure conformity with the corresponding frameworks and technologies.

Using facets enables you to download and configure the necessary framework components, have various descriptors generated automatically and stored in proper locations, and so on.

Most facets may be added to a module irrespective of whether other facets exist.

There are also facets that extend other facets. Such facets are possible only if a module has their parent facet.

翻译：

facets代表了模块中使用的各种框架，技术，语言。他们让idea知道如何对待这些模块的上下文，以及对框架和技术处理的一致性。

通过使用facets可以使你下载并配置必须的框架要素，自动生成各种描述符，并存储在合适的路径下等。

大多数facets可以被添加在模块中而不管其他facets是否存在。

facets也可以扩展其他的facets。如果一个模块有父facet，就可能出现这种情况。

#### Artifacts
An artifact in IntelliJ IDEA may mean two different but related things.

First of all, an artifact is a specification of the output that you want to be generated for your project.

In this sense, the artifact is short for artifact configuration or artifact specification.

In the second sense, an artifact is the actual output generated according to the corresponding specification.

An artifact may be as simple as the compilation output for one or more of your modules packaged in a Java archive (JAR).

An artifact may be a Web application archive (WAR) or an enterprise archive (EAR), packaged or exploded.

In that case, the artifact will normally include Web or Java EE Application facet resources such as deployment descriptors. Other artifact formats are also available.

In many of the cases, the generated artifacts are deployment-ready.

That is, most of the artifact formats offered by IntelliJ IDEA are suitable for deployment in corresponding target software environments (runtimes, platforms, application servers, etc.)

You can define and then generate as many artifacts as you want.

To conclude, the artifacts let you combine your compiled source code, associated libraries, metadata and resources (text, images and so on) in deployable units.

They may include compilation results for more than one module as well as facet resources such as deployment descriptors and other configuration files.

In addition, artifacts can include other artifacts.

翻译：

idea中的artifact中有两种含义。

首先一个artifact是你想通过你的工程产生特定的输出，这种场景下，一个artifact是artifact配置，或者artifact详述的缩写。

第二个场景是一个artifact是工程根据你的配置真实的输出。

一个artifcat可以是你的一个或多个的模块的简单的编译输出打成的jar包。

也可以是一个web应用打的war包，或者一个EAR，打包或者直接部署。

这种情况下，artifact正常情况下包含web或者java EE应用的facet资源，如部署脚本。一些别的格式的artifact也是可以的。

大多数情况下，生成的artifacts是准备部署的，也就是说，大多数的idea提供的artifacts格式是适合部署到特定的软件环境中（运行时，平台，应用服务器等）

你也可以自己定义自己想要的artifact。

结论：artfacts让你可以结合你部署单元中的编译好的源码，相关的类库，metadata，资源（文本，图片等），他们可以包含多个模块的编译结果，facet资源如部署描述和其他的配置文件等

另，artfacts可以包含其他的artifacts。