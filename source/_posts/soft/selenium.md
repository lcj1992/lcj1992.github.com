---
layout: post
title: selenium
date: 2015-04-20
categories: soft
tags: selenium
---

### selenium环境搭建  

*	java环境
*	下载Selenium IDE、SeleniumRC、IEDriverServer
*	 启动SeleniumRC

### selenium几个工具的联系

*	Selenium Core：支持DHTML的测试案例（效果类似数据驱动测试），它是Selenium  IDE和Selenium  RC的引擎
*	Selenium IDE （firefox上一个脚本录制工具）
*	Selenium RC（Selenium Remote Control,后来被WebDriver取代，）
*	Selenium Grid：允许同时并行地、在不同的环境上运行多个测试任务，极大地加快Web应用的功能测试。
*	IEDriverServer（IE驱动）

### 选择需要考虑的问题

**浏览器支持**  
*	IDE 仅可以在firefox中工作
*	RC支持很多
*	Core所有  
  
**是否需要远程安装**  
*	只有Core需要  
  
**多语言支持**    
*	IDE,Core仅支持Selenium语言  
*	RC（WebDriver）支持很多语言，java  
  
**Selenium语言**    
Command，Target，Value三种元素组成一个行为，并且有辅助录制脚本的工具（如Selenium   IDE），但是他没有条件，循环，这会使复杂的测试编的困难甚至不可能。    
   
**结论**   
建议使用Selenium IDE + FireBug进行测试案例的编写，然后转为其他语言的测试案例后，再调用Selenium RC运行测试案例。

### webDriver常用语法
WebDriver driver = new FirefoxDriver();


参考

[1]<https://selenium.googlecode.com/git/docs/api/java/index.html>

[2]<https://code.google.com/p/selenium/wiki/GettingStarted>

[3]<https://www.ibm.com/developerworks/cn/web/1306_chenlei_webdriver/>

[4]<http://stackoverflow.com/questions/6430462/how-to-select-get-drop-down-option-in-selenium-2>

[5]<http://www.ibm.com/developerworks/cn/java/j-lo-keyboard/>
