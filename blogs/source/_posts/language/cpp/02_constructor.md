---
title: 02 constructor
tags:
categories:
- language
- cpp
---

## constructor
``` cpp
	#include <iostream>
	
	using namespace std;
	
	namespace Constructor
	{
	
	    class Student
	    {
	    private:
	        const char *m_name; // 初始化 const 成员变量的唯一方法就是使用初始化列表
	        int m_age;
	        float m_score;
	
	    public:
	        /**
	     * 一个类必须有构造函数，要么用户自己定义，要么编译器自动生成。
	     * 一旦用户自己定义了构造函数，不管有几个，也不管形参如何，编译器都不再自动生成。
	     */
	        Student(char *name, int age, float score);
	        void show();
	    };
	
	    Student::Student(char *name, int age, float score) : m_name(name), m_age(age) // 初始化列表
	    {
	        this->m_score = score;
	    }
	
	    void Student::show()
	    {
	        cout << this->m_name << "的年龄是: " << this->m_age << ", 成绩是: " << this->m_score << endl;
	    }
	} // namespace Constructor
	
	using namespace Constructor;
	// main函数不能再namespace里面
	int main()
	{
	    // 有了上面的using namespace constructor, 接下来就可以加上constructor::也可以不加上constructor::
	    Constructor::Student *stu = new Student("李华", 10, 100);
	    stu->show();
	
	    system("pause");
	    return 0;
	}
```


