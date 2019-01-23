---
layout: post
title: 回调函数与函数指针
date: 2015-01-25
categories: language
description: function_pointer
---


## 函数指针
>指针 地址，函数指针既是函数地址，函数名也是一个函数地址，是一个函数指针常量  

下边c++虚函数表的解析陈皓的博文中也涉及到函数指针的内容，以此为例分析一下函数指针吧。

	`class Base{
	public：
		virtual void f(){cout<<"Base::f"<<endl;}
		virtual void g(){cout<<"Base::g"<<endl;}
		virtual void h(){cout<<"Base::h"<<endl;}
	};

	//然后我们Base的实例来得到虚函数表
	typedef void(*Fun)(void);
	Base b;
	Fun pFun = NULL;
	cout<<"虚函数表地址："<<(int*)(&b)<<endl;
	cout<<"虚函数表-第一个函数地址："<<(int*)*(int*)(&b)<<endl;
	pFun = (Fun)*((int*)*(int*)(&b));
	//pFun = (Fun)*((int*)*(int*)(&b) + 0);//Base::f
	//pFun = (Fun)*((int*)*(int*)(&b) + 1);//Base::g
	//pFun = (Fun)*((int*)*(int*)(&b) + 2);//Base::h
	pFun();`

*	`typedef void(*Fun)(void);`  
	Fun是一个参数为空，返回值为空的函数指针。
*	`pFun = (Fun)*((int*)*(int*)(&b) + 0);`  
	`&b`取得实例的地址，  
	`(int *)(&b)`做强转化，取得虚函数表的地址，  
	`*(int *)(&b)`取虚函数表中第一个函数地址的地址，  
	`((int*)*(int*)(&b) + 0)`加0也即虚函数表中第一个函数地址的地址，  
	`*((int*)*(int*)(&b) + 0)`取得函数的入口地址，  
	`(Fun)*((int*)*(int*)(&b) + 0)`强制类型转化，将函数入口地址转化成Fun类型函数指针。
	`pFun`最后通过pFun()调用虚函数表中第一个函数。  

## 回调函数
>回调函数：一个函数的参数是另一个函数的地址。
例子：

	void print(int test()){
		printf("%d\n", test());
	}

	int return1(){
		return 1;
	}
	int return2(){
		return 2;
	}

	int main(void)
	{
		print(&return1);
		print(&return2);
		getchar();
	}
