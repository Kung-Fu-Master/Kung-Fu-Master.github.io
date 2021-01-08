---
title: 01 const new&delete inline overload externC
tags:
categories:
- language
- cpp
---

## c++ base
头文件, const_newDelete_inline_overload_externC.h

``` cpp
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
```

源文件 const_newDelete_inline_overload_externC.cpp

``` cpp
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
```

编译：

``` shell
	g++ const_newDelete_inline_overload_externC.cpp -I ./
```

执行

``` shell
	$ ./a.out
	10
	2
	1 11.9 @
	1 1.2 @
	1 1.3 !
```

## string类
在C语言中，有两种方式表示字符串：
 * 一种是用字符数组来容纳字符串，例如char str[10] = "abc"，这样的字符串是可读写的；
 * 一种是使用字符串常量，例如char *str = "abc"，这样的字符串只能读，不能写。
两种形式总是以\0作为结束标志.

### copy-on-write
只有当字符串被修改的时候才创建各自的拷贝，这种实现方式称为写时复制（copy-on-write）策略.  
当字符串只是作为值参数（value parameter）或在其他只读情形下使用，这种方法能够节省时间和空间.  

``` cpp
	string s1("12345");
	string s2 = s1;
	cout << (s1 == s2) << endl;
	s1[0] = '6';
	cout << "s1 = " << s1 << endl;  //62345
	cout << "s2 = " << s2 << endl;  //12345
	cout << (s1 == s2) << endl;
```

在 GCC 下的运行结果:

```
	1
	s1 = 62345
	s2 = 12345
	0
```

### length
string 是 C++ 中常用的一个类.  
与C风格的字符串不同，string 的结尾没有结束标志'\0'.  

``` cpp
	string s = "http://c.biancheng.net";
	int len = s.length();
	cout<< len <<endl; // 输出 8
```

### c_str()
在实际编程中，有时候必须要使用C风格的字符串（例如打开文件时的路径），为此，string 类为我们提供了一个转换函数 c_str()
c_str()能够将 string 字符串转换为C风格的字符串，并返回该字符串的 const 指针（const char*）

``` cpp
	string path = "D:\\demo.txt";
	FILE *fp = fopen(path.c_str(), "rt");
```

### cin >> 输入字符串 string
可以像对待普通变量那样对待 string 变量，也就是用>>进行输入，用<<进行输出.  
``` cpp
	string s;
	cin >> s;  //输入字符串
	cout << s << endl;  //输出字符串
```
运行结果：
```
	http://c.biancheng.net  http://vip.biancheng.net↙
	http://c.biancheng.net
```
虽然我们输入了两个由空格隔开的网址，但是只输出了一个，这是因为输入运算符>>默认会忽略空格，遇到空格就认为输入结束，所以最后输入的http://vip.biancheng.net没有被存储到变量 s。

### 访问字符串中的字符
```cpp
	string s = "1234567890";
	for(int i=0,len=s.length(); i<len; i++){
	    cout<<s[i]<<" ";
	}
	cout<<endl;
	s[5] = '5';
	cout<<s<<endl;
```
运行结果：
```
	1 2 3 4 5 6 7 8 9 0
	1234557890
```
### 字符串的拼接
可以使用+或+=运算符来直接拼接字符串, 再也不需要使用C语言中的 strcat()、strcpy()、malloc() 等函数来拼接字符串了，再也不用担心空间不够会溢出了.  
用+来拼接字符串时，运算符的两边可以都是 string 字符串，也可以是一个 string 字符串和一个C风格的字符串，还可以是一个 string 字符串和一个字符数组，或者是一个 string 字符串和一个单独的字符.  
``` cpp
	string s1 = "first ";
	string s2 = "second ";
	char *s3 = "third ";
	char s4[] = "fourth ";
	char ch = '@';
	string s5 = s1 + s2;
	string s6 = s1 + s3;
	string s7 = s1 + s4;
	string s8 = s1 + ch;
	
	cout<<s5<<endl<<s6<<endl<<s7<<endl<<s8<<endl;
```
运行结果：
```
	first second
	first third
	first fourth
	first @
```
### string 字符串的增删改查

#### insert()
insert() 函数可以在 string 字符串中指定的位置插入另一个字符串，它的一种原型为：
``` cpp
	string& insert (size_t pos, const string& str);
```
pos 表示要插入的位置，也就是下标；str 表示要插入的字符串，它可以是 string 字符串，也可以是C风格的字符串。  
实例:
``` cpp
	string s1 = "1234567890";
	string s2 = "aaa";
	s1.insert(5, s2);
```
运行结果：
```
	12345aaa67890
```
#### erase()
erase() 函数可以删除 string 中的一个子字符串。它的一种原型为：
``` cpp
	string& erase (size_t pos = 0, size_t len = npos);
```
pos 表示要删除的子字符串的起始下标，len 表示要删除子字符串的长度。如果不指明 len 的话，那么直接删除从 pos 到字符串结束处的所有字符（此时 len = str.length - pos）。

#### substr()
substr() 函数用于从 string 字符串中提取子字符串，它的原型为：
``` cpp
	string substr (size_t pos = 0, size_t len = npos) const;
```
pos 为要提取的子字符串的起始下标，len 为要提取的子字符串的长度。

#### find() 函数
find() 函数用于在 string 字符串中查找子字符串出现的位置，它其中的两种原型为：
``` cpp
	size_t find (const string& str, size_t pos = 0) const;
	size_t find (const char* s, size_t pos = 0) const;
```
第一个参数为待查找的子字符串，它可以是 string 字符串，也可以是C风格的字符串。第二个参数为开始查找的位置（下标）；如果不指明，则从第0个字符开始查找.  

#### rfind() 函数
rfind() 和 find() 很类似，同样是在字符串中查找子字符串，不同的是 find() 函数从第二个参数开始往后查找，而 rfind() 函数则最多查找到第二个参数处，如果到了第二个参数所指定的下标还没有找到子字符串，则返回一个无穷大值4294967295.  

#### find_first_of() 函数
find_first_of() 函数用于查找子字符串和字符串共同具有的字符在字符串中首次出现的位置









