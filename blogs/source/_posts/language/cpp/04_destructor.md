---
title: 04 Destructor
tags:
categories:
- language
- cpp
---

### destructor

### **构造函数和析构函数执行顺序**
类对象生成时，先执行所有成员对象的构造函数，然后才执行类自己的构造函数。成员对象构造函数的执行次序和成员对象在类定义中的次序一致，与它们在构造函数初始化列表中出现的次序无关.  
当类对象消亡时，先执行类的析构函数，然后再执行成员对象的析构函数，成员对象析构函数的执行次序和构造函数的执行次序相反，即先构造的后析构.  

``` cpp
	#include <iostream>
	using namespace std;
	
	/**
	 * 析构函数（Destructor）也是一种特殊的成员函数，没有返回值.
	 * 不需要程序员显式调用（程序员也没法显式调用）而是在销毁对象时自动执行.
	 * 构造函数的名字和类名相同，而析构函数的名字是在类名前面加一个~符号.
	 * 析构函数没有参数，不能被重载，因此一个类只能有一个析构函数.
	 * 如果用户没有定义，编译器会自动生成一个默认的析构函数。
	 */
	
	namespace Destructor
	{
	    class VLA
	    {
	    public:
	        // 用 new 分配内存时会调用构造函数
	        VLA(int len);
	        // 用 delete 释放内存时会调用析构函数
	        ~VLA();
	
	    private:
	        const int m_len; // m_len 变量不允许修改, 只能通过初始化列表赋值.
	        int *m_arr;
	        int *m_p;
	
	    public:
	        void input();
	        void show();
	
	    private:
	        int *at(int i); // at() 函数只在类的内部使用
	    };
	
	    VLA::VLA(int len) : m_len(len)
	    {
	        if (len > 0)
	        {
	            // new 创建的对象位于堆区，通过 delete 删除时才会调用析构函数
	            // 如果没有 delete，析构函数就不会被执行.
	            m_arr = new int[len];
	        }
	        else
	        {
	            m_arr = NULL;
	        }
	    }
	
	    VLA::~VLA()
	    {
	        delete[] m_arr;
	        cout << "Delete m_arr" << endl;
	    }
	
	    void VLA::input()
	    {
	        for (int i = 0; m_p = at(i); i++)
	        {
	            cin >> *at(i);
	            cout << "&mp: " << m_p << endl;
	        }
	    }
	
	    void VLA::show()
	    {
	        for (int i = 0; m_p = at(i); i++)
	        {
	            if (i == m_len - 1)
	            {
	                cout << *at(i) << endl;
	            }
	            else
	            {
	                cout << *at(i) << ", ";
	            }
	        }
	    }
	
	    int *VLA::at(int i)
	    {
	        if (!m_arr || i < 0 || i >= m_len)
	        {
	            return NULL;
	        }
	        else
	        {
	            return m_arr + i;
	        }
	    }
	} // namespace Destructor
	
	using namespace Destructor;
	int main()
	{
	    int n;
	    cout << "Input array length: ";
	    cin >> n;
	    VLA *parr = new VLA(n);
	    parr->input();
	    parr->show();
	    delete parr;
	    return 0;
	}
```
## **输出**
```
	Input array length: 3 
	1 2 3
	&mp: 0x11f5030
	&mp: 0x11f5034
	&mp: 0x11f5038
	1, 2, 3
	Delete m_arr
```

