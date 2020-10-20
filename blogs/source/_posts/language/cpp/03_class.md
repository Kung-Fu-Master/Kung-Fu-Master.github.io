---
title: 03 class
tags:
categories:
- language
- cpp
---

## 类class

	#include <iostream>
	using namespace std;

### 普通类

	/**
	 * 类只是一个模板（Template），编译后不占用内存空间，不存在于编译后的可执行文件中；
	 * 所以在定义类时不能对成员变量进行初始化，因为没有地方存储数据。
	 * 只有在创建对象以后会在栈区或者堆区分配内存，这个时候就可以赋值了。
	 *
	 */
	class Student
	{
	    //C++ 中的 public、private、protected 只能修饰类的成员，不能修饰类，C++中的类没有共有私有之分.
	private:
	    // name、age、score 在内存地址分布按照声明的顺序由低到高依次排列，和结构体非常类似，也会有内存对齐的问题.
	    char *name;
	    int age;
	    float score;
	
	public:
	    // 在类体内部对成员函数作声明，而在类体外部进行定义，这是一种良好的编程习惯，实际开发中大家也是这样做的.
	    void say();
	    void set_variable(char *name, int age, float score);
	}; // 类定义的最后有一个分号;，它是类定义的一部分，表示类定义结束了，不能省略。
	
	void Student::say()
	{
	    cout << this->name << "的年龄是: " << this->age << ", 成绩是: " << this->score << endl;
	}
	
	void Student::set_variable(char *name, int age, float score)
	{
	    this->name = name;
	    this->age = age;
	    this->score = score;
	}
### **封闭类**

	/**
	 *  一个类的成员变量如果是另一个类的对象，就称之为“成员对象”。包含成员对象的类叫封闭类（enclosed class）
	 * 创建封闭类的对象时，它包含的成员对象也需要被创建，这就会引发成员对象构造函数的调用。
	 * 如何让编译器知道，成员对象到底是用哪个构造函数初始化的呢？这就需要借助封闭类构造函数的初始化列表
	 */
	class Tyre // 轮胎类
	{
	public:
	    Tyre(int radius, int width);
	    void show() const;
	
	private:
	    int m_radius;
	    int m_width;
	};
	Tyre::Tyre(int radius, int width) : m_radius(radius), m_width(width)
	{
	    cout << "Tyre constructor" << endl;
	}
	Tyre::~Tyre()
	{
	    cout << "Tyre destructor" << endl;
	}
	void Tyre::show() const
	{
	    cout << "轮胎半径:" << this->m_radius << ", 轮胎宽度:" << this->m_width << endl;
	}
	
	class Engine // 引擎类
	{
	public:
	    // 设置了默认参数的构造函数可以看作无参构造函数
	    Engine(float displacement = 2.0);
	    void show() const;
	
	private:
	    float m_displacement;
	};
	Engine::Engine(float displacement) : m_displacement(displacement)
	{
	    cout << "Engine constructor" << endl;
	}
	Engine::~Engine()
	{
	    cout << "Engine destructor" << endl;
	}
	void Engine::show() const
	{
	    cout << "排量: " << this->m_displacement << "L" << endl;
	}
	
	class Car
	{
	public:
	    Car(int price, int radius, int width);
	    void show() const;
	
	private:
	    int m_price;
	    Tyre m_tyre;
	    Engine m_engine;
	};
	// m_engine 默认用 Engine 类的无参构造函数初始化
	Car::Car(int price, int radius, int width) : m_price(price), m_tyre(radius, width)
	{
	    cout << "Car constructor" << endl;
	}
	Car::~Car()
	{
	    cout << "Car destructor" << endl;
	}
	void Car::show() const
	{
	    cout << "价格: " << this->m_price << endl;
	    this->m_tyre.show();
	    this->m_engine.show();
	}

### **对象数组**

	/**
	 * 对象数组
	 */
	class CTest
	{
	public:
	    CTest(); //不带花括号是声明, 带花括号则是声明+定义就不能在类外重新定义构造函数, 其它函数也一样.
	    CTest(int n);
	    CTest(int n, int m);
	};
	
	CTest::CTest()
	{
	    cout << "Constructor 0 Called!" << endl;
	}
	
	CTest::CTest(int n)
	{
	    cout << "Constructor 1 Called!" << endl;
	}
	
	CTest::CTest(int n, int m)
	{
	    cout << "Constructor 2 Called!" << endl;
	}

### **静态成员变量**
使用静态成员变量来实现多个对象共享数据的目标。静态成员变量是一种特殊的成员变量，它被关键字static修饰.  
共享数据的典型使用场景是计数.  
 * static 成员变量属于类，不属于某个具体的对象，即使创建多个对象，也只为static变量分配一份内存，所有对象使用的都是这份内存中的数据.  
 * static 成员变量必须在类声明的外部初始化.  
 * 静态成员变量在初始化时不能再加 static，但必须要有数据类型。被 private、protected、public 修饰的静态成员变量都可以用这种方式初始化.  
 * static 成员变量不占用对象的内存，而是在所有对象之外开辟内存，即使不创建对象也可以访问.  
**注意：** static 成员变量的内存既不是在声明类时分配，也不是在创建对象时分配，而是在（类外）初始化时分配。反过来说，没有在类外初始化的 static 成员变量不能使用


	class Student1
	{
	public:
	    Student1(char *name, int age, float score);
	    void show();
	
	public:
	    static int m_total; //静态成员变量
	private:
	    char *m_name;
	    int m_age;
	    float m_score;
	};
	int Student1::m_total = 0;
	Student1::Student1(char *name, int age, float score) : m_name(name), m_age(age), m_score(score)
	{
	    m_total++; //操作静态成员变量
	}
	void Student1::show()
	{
	    cout << m_name << "的年龄是" << m_age << "，成绩是" << m_score << "（当前共有" << m_total << "名学生）" << endl;
	}

### **静态成员函数**
普通成员变量占用对象的内存，静态成员函数没有 this 指针，不知道指向哪个对象，无法访问对象的成员变量，也就是说静态成员函数不能访问普通成员变量，只能访问静态成员变量.  
普通成员函数必须通过对象才能调用，而静态成员函数没有 this 指针，无法在函数体内部访问某个对象，所以不能调用普通成员函数，只能调用静态成员函数。  
`静态成员函数与普通成员函数的根本区别在于：普通成员函数有 this 指针，可以访问类中的任意成员；而静态成员函数没有 this 指针，只能访问静态成员（包括静态成员变量和静态成员函数）.`  

	class Student2
	{
	public:
	    Student2(char *name, int age, float score);
	    void show();
	
	public: //声明静态成员函数
	    static int getTotal();
	    static float getPoints();
	
	private:
	    static int m_total;    //总人数
	    static float m_points; //总成绩
	private:
	    char *m_name;
	    int m_age;
	    float m_score;
	};
	
	int Student2::m_total = 0;
	float Student2::m_points = 0.0;
	Student2::Student2(char *name, int age, float score) : m_name(name), m_age(age), m_score(score)
	{
	    m_total++;
	    m_points += score;
	}
	void Student2::show()
	{
	    cout << m_name << "的年龄是" << m_age << "，成绩是" << m_score << endl;
	}
	//定义静态成员函数
	int Student2::getTotal()
	{
	    return m_total;
	}
	float Student2::getPoints()
	{
	    return m_points;
	}

### *const 成员函数,变量 和 const 对象*
const 成员函数可以使用类中的所有成员变量，但是不能修改它们的值，这种措施主要还是为了保护数据而设置的。const 成员函数也称为常成员函数.  
常成员函数需要在声明和定义的时候在函数头部的结尾加上 const 关键字.  
`需要强调的是，必须在成员函数的声明和定义处同时加上 const 关键字`
 * 函数开头的 const 用来修饰函数的返回值，表示返回值是 const 类型，也就是不能被修改，例如const char * getname()。
 * 函数头部的结尾加上 const 表示常成员函数，这种函数只能读取成员变量的值，而不能修改成员变量的值，例如char * getname() const。

const 也可以用来修饰对象，称为常对象。一旦将对象定义为常对象之后，就只能调用类的 const 成员（包括 const 成员变量和 const 成员函数）.  


	class Student3{
	public:
	    Student3(char *name, int age, float score);
	    void show();
	    //声明常成员函数
	    char *getname() const;
	    int getage() const;
	    float getscore() const;
	private:
	    char *m_name;
	    int m_age;
	    float m_score;
	};
	Student3::Student3(char *name, int age, float score): m_name(name), m_age(age), m_score(score){ }
	void Student3::show(){
	    cout<<m_name<<"的年龄是"<<m_age<<"，成绩是"<<m_score<<endl;
	}
	//定义常成员函数
	char * Student3::getname() const{
	    return m_name;
	}
	int Student3::getage() const{
	    return m_age;
	}
	float Student3::getscore() const{
	    return m_score;
	}

### **友元函数**
借助友元（friend），可以使得其他类中的成员函数以及全局范围内的函数访问当前类的 private 成员.  
 * 在当前类以外定义的、不属于当前类的函数也可以在类中声明，但要在前面加 friend 关键字，这样就构成了友元函数.  
 * 友元函数可以是不属于任何类的非成员函数，也可以是其他类的成员函数。
 * 友元函数不同于类的成员函数，在友元函数中不能直接访问类的成员，必须要借助对象

#### **非成员友元函数**

	class Student{
	public:
	    Student(char *name, int age, float score);
	public:
	    friend void show(Student *pstu);  //将show()声明为友元函数
	private:
	    char *m_name;
	    int m_age;
	    float m_score;
	};
	Student::Student(char *name, int age, float score): m_name(name), m_age(age), m_score(score){ }
	//非成员函数
	void show(Student *pstu){
	    cout<<pstu->m_name<<"的年龄是 "<<pstu->m_age<<"，成绩是 "<<pstu->m_score<<endl;
	}
	int main(){
	    Student stu("小明", 15, 90.6);
	    show(&stu);  //调用友元函数
	    Student *pstu = new Student("李磊", 16, 80.5);
	    show(pstu);  //调用友元函数
	    return 0;
	}
#### **将其他类的成员函数声明为友元函数**

	// 提前声明Address类
	// 因为在 Address 类定义之前、在 Student 类中使用到了它，如果不提前声明，编译器会报错
	class Address;
	//声明Student类
	class Student{
	public:
	    Student(char *name, int age, float score);
	public:
	    void show(Address *addr);
	private:
	    char *m_name;
	    int m_age;
	    float m_score;
	};
	//声明Address类
	class Address{
	private:
	    char *m_province;  //省份
	    char *m_city;  //城市
	    char *m_district;  //区（市区）
	public:
	    Address(char *province, char *city, char *district);
	    // 将Student类中的成员函数show()声明为友元函数
		// 使得show()利用Address对象来访问Address类的private成员变量
	    friend void Student::show(Address *addr);
	};
	//实现Student类
	Student::Student(char *name, int age, float score): m_name(name), m_age(age), m_score(score){ }
	void Student::show(Address *addr){
	    cout<<m_name<<"的年龄是 "<<m_age<<"，成绩是 "<<m_score<<endl;
	    cout<<"家庭住址："<<addr->m_province<<"省"<<addr->m_city<<"市"<<addr->m_district<<"区"<<endl;
	}
	//实现Address类
	Address::Address(char *province, char *city, char *district){
	    m_province = province;
	    m_city = city;
	    m_district = district;
	}
	int main(){
	    Student stu("小明", 16, 95.5f);
	    Address addr("陕西", "西安", "雁塔");
	    stu.show(&addr);
	   
	    Student *pstu = new Student("李磊", 16, 80.5);
	    Address *paddr = new Address("河北", "衡水", "桃城");
	    pstu -> show(paddr);
	    return 0;
	}

### **友元类**
不仅可以将一个函数声明为一个类的“朋友”，还可以将整个类声明为另一个类的“朋友”，这就是友元类。友元类中的所有成员函数都是另外一个类的友元函数.  
 * 友元的关系是单向的而不是双向的。如果声明了类 B 是类 A 的友元类，不等于类 A 是类 B 的友元类，类 A 中的成员函数不能访问类 B 中的 private 成员。
 * 友元的关系不能传递。如果类 B 是类 A 的友元类，类 C 是类 B 的友元类，不等于类 C 是类 A 的友元类。
`除非有必要，一般不建议把整个类声明为友元类，而只将某些成员函数声明为友元函数，这样更安全一些`  


	class Address;  //提前声明Address类
	//声明Student类
	class Student{
	public:
	    Student(char *name, int age, float score);
	public:
	    void show(Address *addr);
	private:
	    char *m_name;
	    int m_age;
	    float m_score;
	};
	//声明Address类
	class Address{
	public:
	    Address(char *province, char *city, char *district);
	public:
	    //将Student类声明为Address类的友元类
	    friend class Student;
	private:
	    char *m_province;  //省份
	    char *m_city;  //城市
	    char *m_district;  //区（市区）
	};
	//实现Student类
	Student::Student(char *name, int age, float score): m_name(name), m_age(age), m_score(score){ }
	void Student::show(Address *addr){
	    cout<<m_name<<"的年龄是 "<<m_age<<"，成绩是 "<<m_score<<endl;
	    cout<<"家庭住址："<<addr->m_province<<"省"<<addr->m_city<<"市"<<addr->m_district<<"区"<<endl;
	}
	//实现Address类
	Address::Address(char *province, char *city, char *district){
	    m_province = province;
	    m_city = city;
	    m_district = district;
	}
	int main(){
	    Student stu("小明", 16, 95.5f);
	    Address addr("陕西", "西安", "雁塔");
	    stu.show(&addr);
	   
	    Student *pstu = new Student("李磊", 16, 80.5);
	    Address *paddr = new Address("河北", "衡水", "桃城");
	    pstu -> show(paddr);
	    return 0;
	}

### **C++ class和struct区别**
C++ 中保留了C语言的 struct 关键字，并且加以扩充。在C语言中，struct 只能包含成员变量，不能包含成员函数.  
而在C++中，struct 类似于 class，既可以包含成员变量，又可以包含成员函数.  
C++中的 struct 和 class 基本是通用的，唯有几个细节不同：
 * 使用 class 时，类中的成员默认都是 private 属性的；而使用 struct 时，结构体中的成员默认都是 public 属性的。
 * class 继承默认是 private 继承，而 struct 继承默认是 public 继承.
 * class 可以使用模板，而 struct 不能.
在编写C++代码时，强烈建议使用 class 来定义类，而使用 struct 来定义结构体，这样做语义更加明确

## **调用**

	int main()
	{
### **普通类**

	    /**
	     * 栈内存是程序自动管理的，不能使用 delete 删除在栈上创建的对象；
	     */
	    Student stu; // 对象stu在栈上分配内存，需要使用&获取它的地址
	    Student *pstu = &stu;
	    //在栈上创建对象, 16
	    //对象的大小只受成员变量的影响，和成员函数没有关系.
	    cout << sizeof(stu) << endl;
	
	    /**
	     * 通过 new 创建出来的对象就不一样了，它在堆上分配内存，没有名字，只能得到一个指向它的指针，
	     * 所以必须使用一个指针变量来接收这个指针，否则以后再也无法找到这个对象了，更没有办法使用它.
	     * 堆内存由程序员管理，对象使用完毕后可以通过 delete 删除.
	     */
	    Student *pstu1 = new Student; // 在堆上创建对象
	    pstu1->set_variable("小明", 10, 100.0);
	    pstu1->say();
	    //在堆上创建对象, 16
	    cout << sizeof(*pstu1) << endl;
	    //类的大小, 16
	    cout << sizeof(Student) << endl;
	
	    delete pstu1;
#### **输出**

	16
	小明的年龄是: 10, 成绩是: 100
	16
	16
	length: 12

### **封闭类**

	    // m_engine 默认用 Engine 类的无参构造函数初始化
	    Car car(200000, 19, 245);
	    car.show();

### **构造函数和析构函数执行顺序**
封闭类对象生成时，先执行所有成员对象的构造函数，然后才执行封闭类自己的构造函数。成员对象构造函数的执行次序和成员对象在类定义中的次序一致，与它们在构造函数初始化列表中出现的次序无关.  
当封闭类对象消亡时，先执行封闭类的析构函数，然后再执行成员对象的析构函数，成员对象析构函数的执行次序和构造函数的执行次序相反，即先构造的后析构.  

#### **输出**

	Engine constructor
	Tyre constructor
	Car constructor
	价格: 200000
	轮胎半径:19, 轮胎宽度:245
	排量: 2L
	Car destructor
	Tyre destructor
	Engine destructor

### **对象数组**

	// 三个元素分别用构造函数 1、2、0 初始化
	CTest array1[3] = {1, CTest(1, 2)};
	cout << "-----------------" << endl;
	// 三个元素分别用构造函数 2、2、1 初始化
	CTest array2[3] = {CTest(1, 2), CTest(1, 2), 1};
	cout << "-----------------" << endl;
	// 两个元素指向的对象分别用构造函数 1、2 初始化
	// pArray[2] 没有初始化，其值是随机的，不知道指向哪里
	CTest *pArray[3] = {new CTest(4), new CTest(1, 2)};

#### **输出**

	Constructor 1 Called!
	Constructor 2 Called!
	Constructor 0 Called!
	-----------------
	Constructor 2 Called!
	Constructor 2 Called!
	Constructor 1 Called!
	-----------------
	Constructor 1 Called!
	Constructor 2 Called!

### **静态成员变量**

	    // 下面这三种方式是等效的
	    //通过类类访问 static 成员变量
	    Student1::m_total = 10;
	    //通过对象来访问 static 成员变量
	    Student1 stu1("小明", 15, 92.5f);
	    stu1.m_total = 20;
	    //通过对象指针来访问 static 成员变量
	    Student1 *pstu1 = new Student1("李华", 16, 96);
	    pstu1->m_total = 20;
	    Student1::m_total = 0;
	    (new Student1("小明", 15, 90))->show();
	    (new Student1("李磊", 16, 80))->show();
	    (new Student1("张华", 16, 99))->show();
#### **输出**

	小明的年龄是15，成绩是90（当前共有1名学生）
	李磊的年龄是16，成绩是80（当前共有2名学生）
	张华的年龄是16，成绩是99（当前共有3名学生）

### **静态成员函数**

	    (new Student2("小明", 15, 90.6))->show();
	    (new Student2("李磊", 16, 80.5))->show();
	    (new Student2("张华", 16, 99.0))->show();
	    (new Student2("王康", 14, 60.8))->show();
	    int total = Student2::getTotal();
	    float points = Student2::getPoints();
	    cout << "当前共有" << total << "名学生，总成绩是" << points << "，平均分是" << points / total << endl;
	    return 0;
#### **输出**

	小明的年龄是15，成绩是90.6
	李磊的年龄是16，成绩是80.5
	张华的年龄是16，成绩是99
	王康的年龄是14，成绩是60.8
	前共有4名学生，总成绩是330.9，平均分是82.725

### **const 成员变量,函数 和 const 对象**

	    const Student *pstu = new Student("李磊", 16, 80.5);
	    //pstu -> show();  //error
	    cout<<pstu->getname()<<"的年龄是"<<pstu->getage()<<"，成绩是"<<pstu->getscore()<<endl;

	    return 0;
	}










