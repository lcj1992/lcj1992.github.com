---
layout: post
title: 对前端html、css、js的关系的看法
date: 2014-11-04
categories: language
description: 暑期实习接触到前端，根据自己的使用谈一下三者的关系
keywords: Front_end, html, css, javascript
---

  
[html,css,javascript基础知识](http://www.w3school.com.cn/)   
	
	html： 内容  
	javascript：事件方法  
	css:样式(纯粹的个人理解)。      
下边是我对三者之间的关系。
   
*	html 文档对象*  
	*	全局属性  ---> css
	*	事件属性  ---> javascript    
		*	windows事件  
		*	form事件			
		*	keyboard事件	  	
		*	mouse事件  
		*	media事件  

&nbsp;&nbsp;&nbsp;&nbsp;通过*`html`*标签生成文档对象，然后通过*`css`*来控制它的属性：宽高位置颜色等等，通过*`javascript`*来控制它在某事件发生时触发的动作。
2014-11-04-对前端三者关系的看法.md
