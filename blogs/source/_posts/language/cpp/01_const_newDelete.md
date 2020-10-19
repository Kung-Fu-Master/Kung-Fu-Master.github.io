---
title: 01 const new&delete inline overload externC
tags:
categories:
- language
- cpp
---

## c++ base
头文件, const_newDelete_inline_overload_externC.h

	#ifndef CONST_NEWDELETE_INLINE_H
	#define CONST_NEWDELETE_INLINE_H
	#include <iostream>
	
	/**
	* extern "C" 既可以修饰一句 C++ 代码，也可以修饰一段 C++ 代码.
	* 在程序中加上extern "C"后，相当于告诉编译器这部分代码是C语言写的，
	* 因此要按照C语言进行编译，而不是C++
	**/
	#ifdef __cplusplus
	extern "C" void display();
	#else
	void display();
	#endif
	
	#ifdef __cplusplus
	extern "C"
	{
	#endif
	    void func11();
	    void func22();
	#ifdef __cplusplus
	}
	#endif
源文件 const_newDelete_inline_overload_externC.cpp

	#include <iostream>
	#include <const_newDelete_inline_overload_externC.h>
	using namespace std;
	
	inline int SQ(int y) { return y * y; }
	
	/*
	* 默认参数
	* 默认参数只能放在形参列表的最后，而且一旦为某个形参指定了默认值，那么它后面的所有形参都必须有默认值.
	* 在给定的作用域中只能指定一次默认参数
	*/
	float d = 10.8;
	void func1(int n, float b = d + 1.1, char c = '@')
	{
	    cout << n << " " << b << " " << c << endl;
	}
	
	/*函数的重载的规则：
	* 函数名称必须相同。
	* 参数列表必须不同（1.个数不同、2.类型不同、3.参数排列顺序不同等）。
	* 函数的返回类型可以相同也可以不相同。
	* 仅仅返回类型不同不足以成为函数的重载。
	* C++代码在编译时会根据参数列表对函数进行重命名，例如void Swap(int a, int b)会被重命名为_Swap_int_int，void Swap(float x, float y)会被重命名为_Swap_float_float.
	* 从这个角度讲，函数重载仅仅是语法层面的，本质上它们还是不同的函数，占用不同的内存，入口地址也不一样.
	*/
	
	int main()
	{
	    /* const
	     * 全局 const 变量的可见范围是当前文件
	     * C++ 规定，全局 const 变量的作用域仍然是当前文件，但是它在其他文件中是不可见的，这和添加了static关键字的效果类似
	     * 对比 const 和 #define 的优缺点时提到，#define 定义的常量仅仅是字符串的替换，不会进行类型检查，而 const 定义的常量是有类型的，编译器会进行类型检查，相对来说比 #define 更安全，所以鼓励大家使用 const 代替 #define
	     * 
	     */
	    const int n = 10; // C++ 对 const 的处理更像是编译时期的#define，是一个值替换的过程,
	    int *p = (int *)&n;
	    *p = 99;
	    printf("%d\n", n); // printf("%d\n", n);语句在编译时就将 n 的值替换成了 10，效果和printf("%d\n", 10)一样; 输出10
	
	    /* 
	     * new delete
	     */
	    int *int_p1 = (int *)malloc(sizeof(int) * 10); //c语言中, 在堆区分配10个int型的内存空间
	    free(int_p1);                                  //释放内存
	    int *int_p2 = new int[10];                     //c++语言中, 分配10个int型的内存空间
	    delete[] int_p2;                               //释放内存
	    int *int_p3 = new int;
	    delete int_p3;
	
	    /** inline 内联函数在编译时会将函数调用处用函数体替换，编译完成后函数就不存在了，所以在链接时不会引发重复定义错误。
	     *  这一点和宏很像，宏在预处理时被展开，编译时就不存在了。从这个角度讲，内联函数更像是编译期间的宏.
	     *  和宏一样，内联函数可以定义在头文件中（不用加 static 关键字），并且头文件被多次#include后也不会引发重复定义错误.
	     *  宏替换为内联函数
	     *  #define SQ(y) ((y)*(y)) 
	     **/
	    int num = 9;
	    int sq = 200 / SQ(num + 1);
	    cout << sq << endl;
	
	    /*
	     * 默认参数
	     **/
	    func1(1);
	    func1(1, 1.2);
	    func1(1, 1.3, '!');
	
	    return 0;
	}
	
	#endif
编译：

	g++ const_newDelete_inline_overload_externC.cpp -I ./
执行

	$ ./a.out
	10
	2
	1 11.9 @
	1 1.2 @
	1 1.3 !


