---
title: new char[size]()用法
tags:
categories:
- language
- cpp
---

## new char[size]()会初始化并清零字符数组
Reference Link: 
https://stackoverflow.com/questions/41082924/c-difference-between-new-charsize-and-new-charsize
https://blog.csdn.net/huangshanchun/article/details/50008959

	char * text = new char[size];

vs.

	char * text = new char[size]();

new char[size]() would zero-initialize the array. new char[size] would leave it uninitialized, containing random garbage.


在C++ 中new char[]() 编译器默认将其初始化为0，new char[]则不会初始化, 有时候输出会看到垃圾数据。

	#include<iostream>
	using namespace std;
	 
	int main(int argc,char *argv[])
	{
		char *p=new char[10];// vs 编译器则不进行初始化
		char *q=new char[10]();//vs 编译器将其初始化为0
	 
		cout<<"p:"<<p<<endl;
		cout<<"q:"<<q<<endl;
		cin.get();
	}

