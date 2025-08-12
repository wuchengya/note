# 类和对象

c++面向对象的三大特性为：<mark>封装，继承和多态</mark>

## <mark>封装</mark>

### **封装的意义:**

+ 将属性和行为作为一个整体
+ 将属性和行为加以权限控制

### **类的访问权限：**

```c++
class Person
{
public://公共权限：类内可以访问，类外也可以
protected://保护权限：类内可以访问，类外不可以(学习继承的时候会讲)
private://私有权限：类内可以访问，类外不可以
};
```

### struct和class的唯一区别：

+ struct默认权限是public，而class默认权限是private

### **成员属性设置私有：**

+ 可以自己控制读写权限
+ 可以检测写入数据的有效性

```c++
class Person
{
public:
    void setname(string name)
    {
        m_name=name;
    }
    string getname()
    {
        return m_name;
    }
    int getyear()
    {
        return m_year;
    }
    void setidol(idol)
    {
        m_idol=idol;
    }
private:
    string m_name;//可读可写
    int m_year = 18;//只读
    string m_idol;//只写
};
```

## <mark>对象的初始化和清理</mark>

### **构造函数和解析函数**

每个类中都有构造函数和析构函数，当你不提供时，编译器会帮你创建，并且函数为空实现。

+ 构造函数：为对象赋值
+ 析构函数：执行清理工作

构造函数语法：类名(){}

+ 没有返回值也不写void
+ 构造函数可以加入参数，因此可以重载
+ 程序在调用对象的时候会自动调用构造，并且只调用一次

析构函数语法：~类名(){}

+ 没有返回值也不写void
+ 不能发生重载
+ 对象销毁前调用

### **构造函数的分类及调用**

两种分类方式：

+ 按参数：有参构造和无参构造
+ 按类型：普通构造和拷贝构造

三种调用方式

+ 括号法
+ 显示法
+ 隐式转换法

```c++
class Person
{
public:
    int num;
    Person()
    {
        cout<<"无参(默认)构造函数"<<endl;
    }
    Person(int a)
    {
        cout<<"有参构造函数"<<endl;
        num=a;
    }
    Person(const Person &p)
    {
        //拷贝构造函数
    }
};
void invoke_struct_func()
{
    //括号法
    Person p;//如果使用Person p()调用编译器会把它当作函数的声明
    Person p1(1);
    Person p2(p1);
    //显示法
    Person p1 = Person(1);
    Person p2 = Person(p1);
    //隐式转换法
    Person p1 = 1;//Person p1 = Person(1);
    Person p2 = p2;
}
```

### **拷贝构造函数的调用时机**

c++中拷贝构造函数的调用有三种时机：

+ 使用一个已经已经创建完毕的对象来初始化一个新对象
+ 值传递的方式给函数参数传值
+ 以值方式返回局部对象

```c++
class Person
{
public:
    int m_age;
    Person()
    {
        cout<<"默认构造函数的调用\n";
    }
    Person(int age)
    {
         cout<<"参数构造函数的调用\n";
        m_age = age;
    }
    Person(const Person &p)
    {
         cout<<"拷贝构造函数的调用\n";
        m_age = p.m_age;
    }
};
void invoke_1()
{
    Person p1;
    Person p2(p1);
    //p1.m_age = p2.m_age;
}
void dowork(Person p)
{
    
}
void invoke_2()
{
    Person p;
    dowork(p);
}
Person func()
{
    Person p;
    return p;
}
void invoke_3()
{
    Person p1 = func();
}
```

构造函数在类外写

```c++
class girl
{
public:
    girl(int age, string name);
    int m_age;
    string m_name;
};
girl::girl(int age, string name) :m_age(age), m_name(name) {};
```

### **构造函数调用规则**

默认情况下，C++编译器至少给一个类添加3个函数

+ 默认构造函数(无参，函数体为空)
+ 默认析构函数(无参，函数体为空)
+ 默认拷贝构造函数，对属性进行值拷贝

构造函数调用规则如下：

+ 如果用户定义有参构造函数，编译器不会提供默认构造，但会提供默认拷贝构造
+ 如果用户提供拷贝构造函数，编译器将不提供其他构造函数

<mark>有参无无参，有拷贝</mark>

```c++
class Person
{
public:
    int m_age;
    Person(int age)
    {
        m_age = age;
    }
};
void invoke_1()
{
    //Person p;编译器会报错，根据调用规则1，此时类中不存在默认构造函数。
    Person p1(1);
    person p2(p1);
    //p1.m_age = p2.m_age;
    //虽然没有提供拷贝构造函数，但是编译器会给我提供
}
```

<mark>拷贝全都无</mark>

```c++
class Person
{
public:
    int m_age;
    Person(const Person &p)
    {
        m_age = p.m_age;
    }
};
void invoke_2()
{
    Person p1;
    Person p2(2);
    //上面这两种形式都是错的
}
```

### **深拷贝与浅拷贝**

浅拷贝：简单的复制拷贝操作

深拷贝：在堆区重新申请空间，进行拷贝操作

下面是浅拷贝，是错误的测试，原因是同一内存被两次释放。

```c++
class Person
{
public:
    int m_age;
    int* m_hight;
    Person(int age,int hight)
    {
        m_age = age;
        m_hight = new int(hight);
    }
    ~Person()
    {
        if (m_hight != NULL)
        {
            delete m_hight;
         	m_hight = NULL;
        }
    }
};
void wrong_test()
{
    Person p1(1,150);
    Person p2(p1);
    cout << *p2.m_hight << endl;
}
```

下面是深拷贝，是正确的测试。

```c++
class Person
{
public:
    int m_age;
    int* m_hight;
    Person(int age,int hight)
    {
        m_age = age;
        m_hight = new int(hight);
    }
    Person(const Person& p)
    {
        m_age = p.m_age;
        m_hight = new int(*p.m_hight);
    }
    ~Person()
    {
        delete m_hight;
    }
};
void test()
{
    Person p1(1,150);
    Person p2(p1);
}
```

比较：浅拷贝中怕p1.m_hight和p2.m_hight只想同一内存地址，在释放过程中，同一内存被两次释放，造成系统崩溃。

深拷贝中我们重新定义了拷贝函数，并且利用m_hight = new int(*p.m_hight);这一行代码使p1.m_age和p2.m_age指向

了不同的堆区内存，所以在delete时各自释放自己相应位置的内存空间，系统不会崩溃。



### **初始化列表**

c++提供了初始化列表语法，用来初始化属性。

语法：

构造函数():属性1(),属性2(),.....{}

```c++
//利用有参构造灵活赋值
class Person
{
public:
    int m_a,m_b,m_c;
    Person(int a,int b,int c):m_a(a),m_b(b),m_c(c)
    {
        
    }
};
void func()
{
    Person p(5,6,8);
    cout<<p.m_a<<endl;
    cout<<p.m_b<<endl;
    cout<<p.m_c<<endl;
}
```



### **类对象作为类成员**

C++中类的成员可以是另一个类的对象，我们称该成员是对象成员。

```c++
class phone
{
public:
    string m_pname;
    phone(string pname)
    {
        cout << "phone构造函数" << endl;
        m_pname = pname;
    }
    ~phone()
    {
        cout << "phone析构函数" << endl;
    }
};
class Person
{
public:
    string m_name;
    phone m_phone;
    Person(string name, string pname) :m_name(name), m_phone(pname)//这里相当于使用隐式转换: m_phone = pname因此不报错
    {
        cout << "Person构造函数" << endl;
    }
    ~Person()
    {
        cout << "person析构函数" << endl;
    }
};
//当其它类对象作为本类成员时，先构造类对象，在构造自身。析构相反。
```



### **静态成员**

静态成员就是就是在成员变量和成员函数前加上关键字static，称为静态成员。

静态成员分为：

&bull;静态成员变量

* 所有对象共享同一份数据
* 编译阶段内就分配内存
* 类内声明，类外初始化

&bull;静态成员函数

* 所有对象共享一个函数
* 静态成员函数只能访问静态成员变量

```c++
#include <stdio.h>
using namespace std;
void variable_by_visit_object();
void variable_by_visit_class();
void func_by_visit_object();
void func_by_visit_class();
class Person
{
public:
    string name="wcy";
    static int num;//类内声明
    static void func()
    {
        cout<<"这是静态成员函数"<<endl;
        num = 200;
        //name = "txx";这是错误的，静态函数只能操作静态变量。
    }
};
int Person::num = 100;//类外初始化
int main()
{
    variable_by_visit_objict();
    variable_by_visit_cladd();
    return 0;
}
void variable_by_visit_object()
{
    Person p;
    cout<<p.num<<endl;//输出100
    Person p1;
    p1.num = 200;
    cout<<p.num<<endl;//输出200
    //等于说就一份数据，不像其他变量，创建一个类对象就会拷贝一份。
    //因此静态成员不属于任何一个对象，所以他的另一种访问方式不需要经过对象
}
void variable_by_visit_class()
{
    cout<<Person::num<<endl;//通过类名访问。
}


void func_by_visit_object()
{
    Person p;
    p.func();
}
void func_by_visit_class()
{
    Person::func();//通过类名调用。
}
```

## <mark>c++对象模型和this指针</mark>

### **成员变量和成员函数分开储存**

接下来要探讨的是非静态成员和静态成员是否在类中

```c++

class empty
{
    
};
class Person
{
public:
    int num;//调用judge函数，答案是4。说明非静态变量在类内。
    void func(){};//答案是1。说明非静态函数不在类内。
    static int sum;//静态声明
    static void f(){};
    //所有静态成员都不在类内
};
int Person::sum = 1;
void empty_class()
{
    empty em;
    cout<<sizeof(em)<<endl;//答案是1
    //c++编译器会给每个空对象分配一个字节的空间，目的是为了区分不同的空对象。
    //每个空对象都有一个独一无二的内存地址。
}
void judge_class()
{
    Person p;
    cout<<sizeof(p)<<endl;
}
```

<mark>总结：只有非静态变量在类内，其余都不在类内。</mark>

### **this指针概念**

this指针是隐含在每一个非静态成员函数内的一种指针

this指针不需要定义，直接使用即可

this指针的用途：

+ 当形参和成员变量同名时，可用this指针区分
+ 在类的非静态成员函数返回对象本身，可用return *this

```c++
class Person
{
public:
	int age;
	Person(int age)
	{
		age = age;//编译器会认为这两个age都是形参，因此要么使用良好的编程习惯，要么使用this指针
        //this指针用法：this->age = age;
	}
};
void use_1()
{
	Person p1(5);
	cout << p1.age << endl;
}
```

使用场景2：链式编程思想

```c++
class Person
{
public:
	int age;
	Person(int age)
	{
		this->age = age;
	}
	Person& add_age(Person& p)
	{
		this->age += p.age;
		return *this;
	}
};
void use_2()
{
	Person p1(5);
	Person p2(2);
	p2.add_age(p1).add_age(p1);
	cout << p2.age;
}
```

### **空指针访问成员函数**

```c++
class Person
{
public:
    int m_age = 500;
    void showclasssname()
    {
        cout << "name is Person" << endl;
    }
    void showm_age()
    {
        cout << "age is" << m_age << endl;
    }
};
void test()
{
    Person* p = NULL;
    p->showclasssname();
    p->showm_age();//这句代码错误，程序崩溃。原因是空指针在调用m_age时出错。
}
```

### **const修饰成员函数**

常函数：

+ 成员函数加const后，我们称这个函数为常函数
+ 常函数内不可修改成员属性
+ 成员属性声明式加关键字mutable后，在常函数中依然可以修改

常对象：

+ 声明对象前加const称该对象为常对象
+ 常对象只能调用常函数

```c++
class Person
{
public:
    int m_age;
    mutable int m_height;
    //这里使用了this指针，在成员函数后面加上const后，this指针指向的值也不能修改。
    //加上const的函数被称为常函数。
    void set_age() const
    {
        //m_age = 18;操作不当
        //那如何改变呢？使用关键字mutable,即使是在常函数中也能够修改变量
        m_height = 173;
    }
};
```

```c++
class Person
{
public:
    mutable int m_age;
    int m_height;
    void set_age() const
    {
        m_age = 18;
    }
    void common_func()
    {
        m_height = 173;
    }
};
void test()
{
    //这是常对象
    const Person p;
    p.m_age = 77;
    //p.m_height = 99;报错，原因是m_height是普通变量
    p.set_age();
    //p.common_func();报错，原因是函数为普通函数
}
```

## <mark>友元</mark>

友元的目的就是让一个函数或者类访问另一个类中的私有成员

友元的关键字是friend

友元的三种实现

+ 全局函数做友元
+ 类做友元
+ 成员函数做友元

### **全局函数做友元**

```c++
class house
{
    friend void set_people(house& h);//将全局函数作为值得信任的，可以访问私有属性
private:
    int m_people;
};
void set_people(house &h)
{
    h.m_people = 15;
    cout << h.m_people << endl;
}
void test()
{
    house h;
    set_people(h);
}
```

### **类做友元**

通过将一个类声明为另一个类的友元类，可以使得友元类中的成员函数可以访问被声明为友元的类的私有成员。

友元类的声明通常在类的定义中使用`friend`关键字进行声明。当一个类A被声明为类B的友元类时，类A中的成员函数就

可以访问类B的私有成员。

```c++
#include <iostream>
using namespace std;

class B; // 提前声明类B

class A {
public:
    void displayB(B &b);
};

class B {
private:
    int b_value;
public:
    B(int value) : b_value(value) {}
    friend class A; // 声明类A为友元类
};

void A::displayB(B &b) {
    cout << "B的私有成员值为：" << b.b_value << endl;
}

int main() {
    B myB(42);
    A myA;
    myA.displayB(myB);
    
    return 0;
}
```

友元类虽然提供了访问私有成员的灵活性，但也降低了封装性，因此应该谨慎使用友元类，避免破坏类的封装性。

### **成员函数做友元**

```c++
class building;
class goodgay
{
public:
	building* m_build = new building;
	void visit()
	{
		cout << m_build->sittingroom;
	}
};
class building
{
	friend void goodgay::visit();
public:
	string sittingroom = "客厅";
private:
	string bedroom = "卧室";
};
```

## <mark>**运算符重载**</mark>

运算符重载概念：对已有的运算符重新进行定义，赋予其另一种功能。

### **加号运算符重载**

作用：实现两个自定义类型的数据相加。

注意：运算符可以发生函数重载（其实这一点是必须的，是为了满足不同类型数据之间的加法运算）

+ 成员函数重载加号

```c++
class Person
{
public:
    Person ope(Person& p)//这里如果把ope换成operator+,那么下面就可以直接使用+运算符
    {
        Person temp;
        temp.m_a = this->m_a + p.m_a;
        temp.m_b = this->m_b + p.m_b;
        return temp;
    }
    int m_a;
    int m_b;
};
void test()
{
    Person a;
    Person b;
    a.m_a = 10;
    a.m_a = 10;
    b.m_a = 10;
    b.m_b = 10;
    Person p3 = a.ope(b);//这里可以直接使用a+b
    cout << p3.m_a;
}
```

+ 全局函数重载加号

```c++
class Person
{
public:
    int m_a;
    int m_b;
};
Person operator+(Person &a,Person &b)
{
    Person temp;
    temp.m_a = a.m_a + b.m_a;
    temp.m_b = b.m_b + a.m_b;
    return temp;
}
//这里是运算符重载的函数重载
Person operator+(Person& a, int num)
{
    Person temp;
    temp.m_a = a.m_a + num;
    temp.m_b = a.m_b + num;
    return temp;
}
void test_1()
{
    Person a;
    Person b;
    a.m_a = 10;
    a.m_a = 10;
    b.m_a = 10;
    b.m_b = 10;
    Person c = a + b;
    cout << c.m_a;
}
void test_2()
{
    Person p1;
    a.m_a = 10;
    a.m_b = 10;
    Person d = p1 +100;
    cout<<d.m_a;
}
```

### **左移运算符重载**

作用：可以输出自定义数据类型

一般不会使用成员函数重载左移运算符，因为无法实现cout在左边的情况

```c++
class A
{
public:
	int m_a = 5;
	int m_b = 6;
};
ostream& operator<<(ostream& cout, A& a)
{
	cout << a.m_a << "    " << a.m_b;
	return cout;
}
int main()
{
	A a;
	cout << a << endl;
	return 0;
}
```

### **递增运算符重载**



### **赋值运算符重载**



### **关系运算符重载**



### **函数调用运算符重载**



## <mark>**继承**</mark>

**继承是面向对象的三大特性之一**

继承的好处：

减少类中相同代码的重复

### **继承的基本语法**

class   子类（派生类）  :    继承方式    父类（基类）

问题描述：

假如说有一个几个类，里面都包含有相同的一段长代码，如果在每个类中都写入这段重复的代码，将会浪费时间

这就可以运用到继承的手段

```c++
class cat
{
public:
	void head()
	{
		cout << "一个头" << endl;
	}
	void leg()
	{
		cout << "四条腿" << endl;
	}
};
class lihuacat :public cat//注意这里是public而不是class
{
public:
	string name = "狸花猫";
};
class buoucat :public cat//注意这里是public而不是class
{
public:
	string name = "布偶猫";
};
void test()
{
	lihuacat l_cat;
	l_cat.head();
	l_cat.leg();
	cout << l_cat.name << endl;
	buoucat bu_cat;
	bu_cat.head();
	bu_cat.leg();
	cout << bu_cat.name << endl;
}
```

### **继承方式**

继承方式有三种：

+ public
+ protected
+ private

![](C:\Users\asus\Pictures\Screenshots\继承.png)

<mark>注意：不论是哪种继承方式，都不可能拿到父类中的private数据</mark>

### **继承中的对象模型**

问题：从父类继承中的，哪些属于子类成员？

```c++
class father
{
public:
	int m_a;
protected:
	int m_b;
private:
	int m_c;
};
class son :public father
{
public:
	int m_d;
};
void test()
{
	cout << sizeof(son) << endl;//16
}
```

回答：从父类继承的属性（非静态）不论是public还是private，都会在子类中占有空间，私有属性之所以被访问不到，是因为被编译器隐藏了。

### **继承中的构造和析构顺序**

子类继承父类后，也继承了父类的构造函数和析构函数

问题：父类和子类构造和析构的invoke顺序

```c++
class father
{
public:
    father()
    {
        cout << "father构造" << endl;
    }
    ~father()
    {
        cout << "father析构" << endl;
    }
};
class son :public father
{
public:
    son()
    {
        cout << "son析构" << endl;
    }
    ~son()
    {
        cout << "son析构" << endl;
    }
};
//father构造
//son构造
//son析构
//father析构
//子类被父类包含
```

### **同名成员处理**

问题：如果父类和子类中出现了相同名称的属性和函数？如何调用自己想调用的哪个呢?

```c++
class father
{
public:
    int m_num = 1;
    void func()
    {
        cout << "same father" << endl;
    }
};
class son :public father
{
public:
    int m_num = 2;
    void func()
    {
        cout << "same son" << endl;
    }
};
void test_1()//成员属性（函数）测试
{
    son s;
    cout << s.m_num << endl;//运行后会打印2，是子类中的数据。
    //那么如何输出father中的m_num呢？
    //只需要在m_num的前面加入father::即可
    cout << s.father::m_num << endl;
}
void test_2()//成员函数测试
{
    son s1;
    s1.func();
    s1.father::func();
    //和成员属性是一样的道理
}
```

回答：如果想要invoke子类的话，正常写

if you want to invoke father class :在函数名字前面加上father::

<mark>注意：</mark>

+ <mark>如果父类中有相同名字的重载函数，也是要正常使用invoke父类的方法，因为当子类中出现和父类中相同名字的属性或者函数时，父类中的会被隐藏</mark>

### **同名静态成员处理**

answer：

静态成员和非静态成员出现同名，处理方式几乎相同

但是静态成员比较特殊的就是可以不通过对象invoke，可以通过类名直接invoke

下面只讲述这种情况

```c++
class father
{
public:
    static int m_num;
    static void func()
    {
        cout << "father static" << endl;
    }
};
int father::m_num = 1;
class son :public father
{
public:
    static int m_num;
    static void func()
    {
        cout << "son static" << endl;
    }
};
int son::m_num = 2;
void test()
{
    //静态成员属性测试
    cout << son::m_num << endl;
    cout << son::father::m_num << endl;
    //其实这里可以把第类的名字当作入口，表示要进入哪个类
    //就如第二个案例，从son进入，在进入father，找到m_num，这即为father类下的m_num
    // 当然也可以直接选择进入你想要进入的类 father::m_num
    //静态成员函数测试
    son::func();
    father::func();
    son::father::func();

}
```

### **多继承语法**

c++允许一个类继承多个类

语法：class 子类：继承方式 father_1 , 继承方式 father_2

多继承可能会引发父类中有同名成员出现，需要加以作用域区分

c++实际开发中不建议使用多继承

```c++
class father_1
{
public:
    int m_a;
    father_1()
    {
        m_a = 1;
    }
};
class father_2
{
public:
    int m_a = 2;
};
class son:public father_1,public father_2
{
};
void test()
{
    son s;
    cout << s.father_1::m_a << endl;
    cout<<s.father_2::m_a<<endl;
}
```

### **菱形继承**

菱形继承概念：

两个派生类继承同一个基类，同时存在一个类同时继承这两个派生类

实例：

比如狮虎兽，假如说基类是动物特征，两个派生类是狮子类和老虎类，最后一个狮虎兽继承了狮子类和老虎类

那么这就回导致一个问题，狮子和老虎都从基类继承了同一份数据，但是狮虎兽只需要一份，如何解决呢？

可以使用虚继承

```c++
class animal
{
public:
    int m_age;
};
class tiger:virtual public animal
{
};
class lion:virtual public animal
{
};
class liger:public tiger,public lion
{
};
void test()
{
    liger li;
    li.m_age = 23;
    cout<<li.m_age<<endl;
    //所以说虚继承可以使两个派生类中的数据变为同一个
    //此时animal类称为虚基类
}
```

## <mark>**多态**</mark>

### **多态的基本概念**

**多态是c++面向对象的三大特性之一**

多态分为两类

+ 静态多态：**函数重载**和**运算符重载**属于静态多态，复用函数名
+ 动态多态：**派生类**和**虚函数**实现运行时多态

静态多态和动态多态的区别：

+ 静态多态的函数地址早绑定 : 编译阶段确定函数地址
+ 动态多态的函数地址晚绑定：运行阶段确定函数地址

动态多态的满足条件：

+ 存在继承关系
+ 子类重写父类的虚函数（子类函数前加不加关键字virtual都可以）

使用方法：

使用父类的指针或者引用传入子类对象

实例：

```c++
class animal
{
public:
	void virtual speak()//虚函数
	{
		cout<<"animal like bark"<<endl;
	}
};
class dog:public animal
{
	void speak()//加不加关键字virtual都可以
	{
		cout<<"dog like bark"<<endl;
	}
};
class cat:public animal
{
	void speak()
	{
		cout<<"cat like bark"<<endl;
	}
};
void dospeak(animal& an)//传入父类指针或者引用
{
	an.speak();
}
void test()
{
	cat c;
	dospeak(c);//传入子类对象
}
```

实际是发生了<mark>对象切片</mark>

什么是对象切片呢？

对象切片是指当子类对象被赋值给父类对象时，子类对象的附加信息会被丢弃，只会保留父类对象的部分。

而子类重写了父类继承的函数，因此会输出重写的函数。

### **纯虚函数和抽象类**

在多态中，通常纯虚函数的实现是毫无意义的，主要都是调用子类重写的内容

因此可以将虚函数改为**纯虚函数**

纯虚函数语法：virtual 返回值类型 函数名  （参数列表） = 0；

当类中有了纯虚函数，这个类也被称为<mark>抽象类</mark>

抽象类特点：

+ 无法实例化对象
+ 子类必须重写抽象类中的纯虚函数，否则也属于抽象类

```c++
class father
{
public:
	//纯虚函数的创建
	virtual void func() = 0;
	
};
class son_1:public father
{
};
class son_2:public father
{
	virtual void func();
};
void test()
{
	//father f;错误，因为抽象类不能实例化，就是不能创建对象
	//son_1 s1;因为类中没有重写纯虚函数，所以此类也为抽象类
	son_2 s2;
}
```

### **虚析构和纯虚析构**

 虚析构函数是在基类中将析构函数声明为虚函数。当基类指针指向派生类对象时，通过基类指针调用析构函数时，会根据

实际对象类型调用相应的析构函数，确保<mark>正确地释放资源</mark>。在基类中声明虚析构函数的方式是在析构函数前面加上

 `virtual` 关键字。

```c++
class base
{
public:
	base()
	{
		cout<<"base结构"<<endl;
	}
	virtual void speak() {cout<<"wcy"<<endl;}
	virtual ~base()
	{
		cout<<"base析构"<<endl;
	}
};
class derive:public base
{
public:
	derive()
	{
		cout<<"derive结构"<<endl;
	}
	void speak() {cout<<"jq"<<endl;}
	~derive()
	{
		cout<<"derive析构"<<endl;
	}
};
void test()
{
	base* ptr = new derive();
	ptr->speak();
	delete ptr;//如果不设置虚析构的话，delete不能正确的释放的内存，就导致内存泄漏
    //也要知道一点，如果内存泄漏的话，析构函数不会被调用
}
```

纯虚析构：

纯虚函数与纯虚析构之间的差别就是纯虚析构一定要有函数体，这是因为如果父类中在堆区也创建了一个指针，那么释放时肯定会使用到析构函数。

注意：当一个类有纯虚析构时，这个类也被称为抽象类，无法实例化对象



# 文件操作

程序运行时产生的数据都属于临时数据，程序一旦运行结束都会被释放

通过文件可以将数据持久化

c++对文件进行操作需要包含头文件<mark><fstream></mark>



文件类型分为两种：

1.文本文件：文件文本以ASKII码的形式储存在计算机中

2.2进制文件：文件文本以二进制的形式储存在计算机中

操作文件的三大类：

1,	ofstream:	写操作

2，	ifstream:	读操作

3，	fstream:	读写操作

## 文本文件

### 写文件

写文件步骤：

1：包含头文件

include<fstream>

2:创建流对象

fstream file;

3:打开文件

file.open("路径"，打开方式)；

4：写入数据

file<<"写入的数据"<<endl;

5:关闭文件

file.close();

文件的打开方式：

| 打开方式    | 解释                       |
| ----------- | -------------------------- |
| ios::in     | 为读文件而打开             |
| ios::out    | 为写文件而打开             |
| ios::ate    | 初始位置：文件尾           |
| ios::app    | 追加方式写文件             |
| ios::trunc  | 如果文件存在先删除，在创建 |
| ios::binary | 二进制方式                 |

注意：可以利用|操作符使用多种文件打开方式

```c++
void test()
{
	fstream file;
	file.open("E:\\c++文件操作专用\\what can i say.txt", ios::out|ios::app);
	if (file.is_open())
	{
		file << "wcy" << endl;
		file.close();
	}
	else
	{
		cout << "文件打开失败" << endl;
	}
}
```

### 读文件

与写文件相比只有读取方式不同

在这里只讲解最好用的一种

```c++
void read_file()
{
	fstream file;
	file.open("E:\\c++文件操作专用\\wcy.txt",ios::in);
	if(file.is_open())
	{
		string str;
		while(getline(file,str))
		{
			cout<<str<<endl;
		}
		
		
		
		file.close();
	}
	else{
		cout<<"file open fail"<<endl;
	}
}
```

注意：在文件操作中，字符串定义最好使用char[字符长度]

## **二进制文件**

二进制文件打开方式要加上ios::binary

而且使用二进制操作的优势是可以写入自定义数据类型

### **写操作**

```c++
class person
{
public:
	string m_name;
	int m_age;
	person(string name,int age);
};
person::person(string name,int age)
{
	m_age = age;
	m_name = name;
}
void write_file()
{
	person p1("wcy",18);
	fstream file;
	file.open("E:\\c++文件操作专用\\what can i say.txt",ios::out|ios::binary|ios::app);
	if(file.is_open())
	{
		cout<<"file was open succssfully"<<endl;
		file.write((const char*)&p1,sizeof(person));
		file<<" "<<endl;
		cout<<"The file was written successfully"<<endl;
		file.close();
		cout<<"The file was closed succssfully"<<endl;
	}
	else{
		cout<<"file was open failed"<<endl;
	}
}
```

### **读文件（有时间补，没搞明白）**

# 模板

## 模板的概念

**模板就是通用的模具，大大提高复用性**

模板的特点：

+ 模板不可以直接使用，它只是一个框架
+ 模板的通用并不是万能的

## 函数模板

+ c++另一种编程思想被称为泛型编程，主要利用的技术就是模板
+ c++提供两种模板机制，**函数模板**和**类模板**

#### 函数模板语法

函数模板作用：

建立一个通用函数，其函数返回值类型和形参类型可以不具体指定，用一个虚拟的类型来代表

语法：

```c++
template<typename T>
函数声明或定义
```

解释：

template ---  声明创建模板

typename --- 表明其后面的符号是一种数据类型，可以用class代替

T --- 通用的数据类型，名称可以替换，通常为大写字母

```c++
void myswap(T& a, T& b)
{
	T temp = a;
	a = b;
	b = temp;
}
void test()
{
	int a = 10;
	int b = 20;
	//1自动类型推导
	myswap(a, b);
	//2显示指定类型
	myswap<int>(a,b);
	cout << a << endl;
	cout << b << endl;
}
```

#### 普通函数与函数模板区别

普通函数可以发生隐式转换。

什么是隐式转换呢？

```c++
int add(int a, int b)
{
	return a + b;
}
char str = 'c';
//我可以将str传入add函数中而不报错，这其实就是编译器将'c'转换为ASKII编码97，这即为隐式转换
template<typename T>
T myadd(T a, T b)
{
	return a + b;
}
//只是一个需要自动类型推导的函数模板
//如果我传入两个不同的数据类型，那么鬼知道T代表什么类型。
那么如果有一个指定类型的函数模板
依然可以发生隐式转换
```

#### 普通函数与函数模板的调用规则

调用规则如下：

+ 函数名字相同，如果函数模板和普通函数都可以实现，优先调用<mark>普通函数</mark>
+ 可以通过空模板参数列表来强制调用函数模板
+ 函数模板也可以发生重载
+ 如果函数模板可以发生更好的匹配，优先调用函数模板

```c++
void my_print(int a, int b)
{
	cout << "invoke common func";
}
template<class T>
void my_print(T a, T b)
{
	cout << "invok template func";
}
void test_1()
{
    my_print(1,2);
}
//调用my_print会调用普通函数
void test_2()
{
    my_print<>(1,2);
}
//空模板参数强制调用

//函数模板也可以发生函数重载
template<typename T>
my_print(T a,T b,T c)
{
    cout<<"invoke overload template func";
}

//what is a better match?
//still the "my_print" function above
//if i pass in two character type parameters(参数)
//But the common function above needs two "int" type parameters
//As the result, the template function will produce a better match
```

#### Limitations of templates

Template can not do everything.

For example,if i pass in "class" type as parameters,the compiler(编译器)will not know how to compare.

We can solve this kind of problems by setting up a specific template.

```c++
class Person
{
public:
    string m_name;
    int m_age;
    Person(string name,int age)
    {
        this->m_name = name;
        this->m_age = age;
    }
};
template<typename T>
bool my_compare(T &a,T &b)
{
    if(a == b) return ture;
    else return false;
}
//The specific operation
//begin
template<> bool my_compare(Person& p1, Person& p2)
{
    if (p1.m_age == p2.m_age && p1.m_name == p2.m_name)
    {
        return true;
    }
    else return false;
}
//end
void test()
{
    Person p1("wcy", 18);
    Person p2("wcy", 18);
    bool result = my_compare(p1, p2);
    if (result)
    {
        cout << "ture";
    }
    else
    {
        cout << "false";
    }
    //Obviously,if i run this code,the compiler will prompt(提示) an error.
    //Other words(也就是说),while "T" = "Person",the compiler do not know how to compare.
    //As a result(therefore),we can list the special case of "T" = "Person" separately(单独地).
    //The specific(具体) operation is to add a special template after a ordinary template.
}
```

summary:

+ We can solve the generalization(通用化) of custom(自定义) type by using a specific templates.
+ Learning templates is not for writing templates,but for being able to use templates provided by the system

at STL.

## Class templates

### The syntax(语法) of class templates.

The role(作用) of class templates:

+ Create a generic class in which the members and data type can be represented by a virtual type without  specifying them.

Syntax:

```c++
template<class T>
class
```

A simple example:

```c++
template<class name_type,class age_type>
class Person
{
public:
	name_type m_name;
	age_type m_age;
	Person(name_type name, age_type age)
	{
		m_age = age;
		m_name = name;
	}
};
void test()
{
	Person<string, int> p1("wcy",999);//If this line of code is changed to Person p1("wcy",999),then the compiler will prompt an error.Class templates can not automatically derive(推导) data type.
	cout << p1.m_name << p1.m_age << endl;
}
```

### The difference between class templates and function templates.

+ Class templates can not automatically derive data type
+  Class templates can initialize(初始化v) a data type

The first case was already covered in the previous lesson.

```c++
template <nametype = string,agetype = int >//We can initialize a data type by this method.
class Person
{
public:
    nametype m_name;
    agetype m_age;
    Person(nametype name,agetype age)
    {
        m_name = name;
        m_age = age;
    }
};
void test()
{
    Person p1("wcy",999);//Here "wcy" and 999 will be given default data type.
}
```

### The creation time of member functions in class templates.

Member function in class templates are created when they are called(调用).

```c++
class Person1
{
public:
	void showPerson1()
	{
		cout << "Person1";
	}
};
class Person2
{
public:
	void showPerson2()
	{
		cout << "Person2";
	}
};
template<class T>
class my_Person
{
public:
	T obj;
	void show1()
	{
		obj.showPerson1();
	}
	void show2()
	{
		obj.shoePerson2();
	}
};
void test()
{
	my_Person<Person1> p;
	p.show1();
}
```

In the example,we create three-class,one is "Person1",the other is "Person2",and the last is a class template "my_Person".

In "test()",we create my_person object "p" and the "T" is given "Person1".When we call "show1()","show1()" will call 

"obj.showPerson1()" . Therefore,if  "obj"  is "Person1"  type,the program will run normally,otherwise(否则),the program

 will crash. 

This means when the compiler compiles this piece of code,it does not check member functions in class 

template,because  "T" is indeterminate(不确定的).

So   member function in class template will be created after compiled. 

### Class templates objects as function parameters.

```c++
template<class T1,class T2>
class Person
{
public:
	T1 m_name;
	T2 m_age;
	Person(T1 name, T2 age)
	{
		m_name = name;
		m_age = age;
	}
	void showperson()
	{
		cout << m_name << endl;
		cout << m_age << endl;
	}
};
//The first way to pass in:(it is also the most commonly used way)
void printPerson_1(Person<string, int> &p)
{
	p.showperson();
}
//The second and the third way of incoming are ridiculous,because they complicate(复杂化) simple thing.
//The second way
template<typename T1,typename T2>
void printPerson_2(Person<T1,T2> &p)
{
    p.showperson();
}

//The third way
template<typename T>
void printPerson_3(T& p)
{
	p.showperson();
}
//There is no need to remember the second and the third way.

void test()
{
	Person<string, int> p("wcy", 999);
	printPerson_1(p);
}
int main()
{
    test();
	return 0;
}
```

### Class template and inheritance(继承)

Here are a few things to keep in mind:

+ When a subclass(子类) inherits from a class template, it needs to specify(指定) the type of T in the parent class when declaring(声明) the subclass.
+  Otherwise, the compiler cannot allocate(分配) memory(有内存的意思) for the subclass.
+  If you want to flexibly specify the type of T in the parent class, the subclass also needs to be a class template.

```c++
template<class T>
class father{};
class son_1 : public father<int>//When a subclass inherris from a class template,you need to specify the type of T in the parent class
{

};
void test()
{
	son_1 s;
}
```

```c++
//If you want to flexibly specify the type of T in the parent class, the subclass also needs to be a class template
template<class T>
class father{};

template<class T1>
class son_1 : public father<T1>
{

};
void test()
{
	son_1<int> s;
}
```

### Implementation(实施) of class template member functions outside of the class

You already understand the class outside implementation of non-template class member functions, just need to add 

scope(作用域) outside.

```c++
template<class T1,class T2>
class Person
{
public:
	T1 m_name;
	T2 m_age;
	Person(T1 name,T2 age);
	void printperson();
};
template<class T1,class T2>
//implementation of the constructor outside the class
Person<T1, T2>::Person(T1 name,T2 age)
{
	m_name = name;
	m_age = age;
}
//implementation of member functions outside the class 
template<class T1, class T2>
void Person<T1, T2>::printperson()
{
	cout << m_name << endl << m_age;
}
```

### Class templates are written in separate files

Because the compiler only compiles functions in class template files at runtime, if we split our class template files into separate files and directly reference(参考) the corresponding(对应的) header files, the compiler will not compile the implementation(实施，实现) of the functions in the source file.

There are two kinds of different solution 

first:

Reference source file directly

second:

Merge(合并) header and source files and change suffix(后缀) to .hpp

The second solution is mainstream.

### Class template and friend

Mastering the implementation of class templates with friend classes both inside and outside the class.

Global function ---implementation inside the class

```c++
template<class p,class q>
class Person
{
	friend void printperson(Person<p,q> p1)//That is a global function
	{
		cout << p1.m_name << p1.m_age;
	}
public:
	Person(p name, q age)
	{
		m_name = name;
		m_age = age;
	}
private:
	p m_name;
	q m_age;
};
void test()
{
	Person<string ,int> p1("aaa", 111);
	printperson(p1);
}
```

Global function ---implementation outside the class

```c++
//We used the "person" class in the following function implementation,so we need to declare the class
template<class p, class q>
class Person;
//When implementing a friend function in a class template, it is recommended to place the function implementation above the class declaration. This is because the member functions and friend functions of a template class need to be declared and defined in the same scope. Placing the friend function implementation below the class declaration may lead to compilation errors as the compiler may not be able to correctly identify the relationship between the friend function and the template class.

//Furthermore, placing the friend function implementation above the class declaration can improve code readability and maintainability. It helps readers better understand the relationship between the friend function and the template class, as well as the specific implementation logic of the friend function.
template<class p, class q>
void printperson(Person<p, q> p1)
{
	cout << p1.m_age << p1.m_name;
}

template<class p,class q>
class Person
{
	friend void printperson<>(Person<p, q> p1);
public:
	Person(p name, q age)
	{
		m_name = name;
		m_age = age;
	}
private:
	p m_name;
	q m_age;
};
void test()
{
	Person<string, int> p11("wcy", 666);
	printperson(p11);
}
```

If you do not have specific requirements ,we do not suggest using the second way.

# STL

## The basic concept of STL

STL从广义上分为：容器，算法，迭代器

容器和算法之间通过迭代器进行无缝连接

STL几乎所有的代码都采用了模板类或者模板函数

## STL六大组件

STL大体分为六大组件，分别是：容器，算法，迭代器，仿函数，适配器，空间配置器

1.容器: 各种数据结构，如vector、list、deque、set、map等,用来存放数据。
2.算法: 各种常用的算法，如sort、find、copy、for_each等
3.迭代器:扮演了容器与算法之间的胶合剂。
4.仿函数:行为类似函数，可作为算法的某种策略。
5.适配器:一种用来修饰容器或者仿函数或迭代器接口的东西。
6.空间配置器:负责空间的配置与管理。

## STL中容器、算法、迭代器

容器:置物之所也
STL容器就是将运用最广泛的一些数据结构实现出来
常用的数据结构:数组,链表,树,栈,队列,集合,映射表 等这些容器分为序列式容器和关联式容器两种:
序列式容器:强调值的排序，序列式容器中的每个元素均有固定的位置。关联式容器:二叉树结构，各元素之间没有严格的物理上的顺序关系

算法:问题之解法也
有限的步骤，解决逻辑或数学上的问题，这一门学科我们叫做算法(Algorithms)算法分为:质变算法和非质变算法。
质变算法:是指运算过程中会更改区间内的元素的内容。例如拷贝，替换，删除等等
非质变算法:是指运算过程中不会更改区间内的元素内容，例如查找、计数、遍历、寻找极值等等。

### vector

存放内置数据类型

```c++
#include<vector>
#include<algorithm>
void print_vector(int value)
{
	cout << value << endl;
}
void test()
{
	vector<int> v;
	v.push_back(10);//尾插法插入数据
	v.push_back(20);
	v.push_back(30);
	v.push_back(40);
	vector<int>::iterator it_begin = v.begin();//拿到第一个元素的地址
	vector<int>::iterator it_end = v.end();//拿到最后一个元素的下一位的地址
	for_each(it_begin, it_end, print_vector);
}
```

存放自定义类型

```c++
class person
{
public:
	string m_name = "wcy";
	int m_age = 111;
};
void test()
{
	person p1;
	person p2;
	vector<person> v;
	v.push_back(p1);
	v.push_back(p2);
	for (vector<person>::iterator it = v.begin();it != v.end();it++)
	{
		cout << it->m_name << it->m_age<<endl;
	}
}

class person
{
public:
	string m_name = "wcy";
	int m_age = 111;
};
void test()
{
	person p1;
	person p2;
	vector<person*> v;
	v.push_back(&p1);
	v.push_back(&p2);
	for (vector<person*>::iterator it = v.begin();it != v.end();it++)
	{
		cout << (*it)->m_name << (*it)->m_age<<endl;
	}
}
```

vector嵌套

```c++
void test()
{
	vector<vector<int>> v;
	vector<int> v1;
	vector<int> v2;
	vector<int> v3;
	for (int i = 0;i < 3;i++)
	{
		v1.push_back(i);
		v2.push_back(i+3);
		v3.push_back(i+6);
	}
	v.push_back(v1);
	v.push_back(v2);
	v.push_back(v3);
	for (vector<vector<int>>::iterator it = v.begin();it != v.end();it++)
	{
		//*it 是vector<int>类型
		for (vector<int>::iterator itint = (*it).begin();itint != (*it).end();itint++)
		{
			cout << *itint << "\t";
		}
		cout << endl;
	}
}
```

## 常用容器

### string

string is essentially(本质上) a class

构造函数：

```c++
string();//创建空字符串
string(const char* s);//使用字符串s初始化
string(const string& str);//使用一个string初始化另一个
string(int n,char c);//使用n个字符初始化
	string s;
	const char* c1 = "hello world";
	char c2[] = "and";//这和字符串string一样了
	string s1(c2);
	string s2 = string(5, 'gfg');//按照最后一个来
	cout << s2;
```

字符串拼接：

```c++
void test()
{
	//运算符拼接
	string s = "我";
	s += "爱玩游戏";
	s += ':';
	string s1 = "LOL WC";
	s += s1;
	cout << s << endl;
	//函数拼接
	string str = "i";
	str.append("love");
	cout << str << endl;
	str.append("game  lll", 4);
	cout << str << endl;
	str.append("LOL", 0, 3);
	cout << str << endl;
}
```

字符串查找和替换：

```c++
string str = "abcdefgde";
int re = str.find("de");//从左往右查找
cout << re << endl;
re = str.rfind("de");//从右往左查找
cout << re;c
```

```c++
string str = "abcdefg";
str.replace(1, 3, "1111");
//从第一个位置三个字符替换为后面的字符串
```

字符串比较：

```c++
string str1 = "xello";
string str2 = "hello";
if (str1.compare(str2) == 0)
{
	cout << "相等" << endl;
}
else if (str1.compare(str2) > 0)
{
	cout << "大于" << endl;
}
else if (str1.compare(str2) < 0)
{
	cout << "小于" << endl;
}
```

string字符存取

```c++
string str = "hello";
for (int i = 0;i < str.size();i++)
{
	cout << str[i] << "  ";
	cout << str.at(i) << " ";
}
str[0] = 'x';
str.at(1) = 'x';
cout << str;
```

string字符插入删除

```c++
string str = "hello";
str.insert(1, "111");
cout << str << endl;
str.erase(1, 3);
cout << str << endl;
```

string求字串

```c++
string str = "abcdef";
string Substr = str.substr(1, 3);
cout << Substr << endl;
```

### vector

构造函数：

```c++
void print_vector(vector<int> &v)
{
	for (vector<int>::iterator it = v.begin();it != v.end();it++)
	{
		cout << *it ;
	}
	cout << endl;
}
void test()
{
	//无参构造
	vector<int> v1;
	for (int i = 0;i < 10;i++)
	{
		v1.push_back(i);
	}
	print_vector(v1);
	//通过区间方式进行构造
	vector<int> v2(v1.begin(), v1.end());
	print_vector(v2);
	//n个元素方式构造
	vector<int> v3(10, 100);
	print_vector(v3);
	//拷贝构造
	vector<int> v4(v3);
	print_vector(v4);
}
```

赋值操作：

```c++
void print_vector(vector<int>& v)
{
	for (vector<int>::iterator it = v.begin();it != v.end();it++)
	{
		cout << *it << "\t";
	}
	cout << endl;
}
void test()
{
	vector<int> v1;
	for (int i = 0;i < 10;i ++ )
	{
		v1.push_back(i);
	}
	print_vector(v1);
    //等号赋值
	vector<int> v2 = v1;
	print_vector(v2);
	vector<int> v3;
    //assign赋值
	v3.assign(v1.begin(), v1.end());
	print_vector(v3);
	vector<int> v4;
	v4.assign(10, 100);
	print_vector(v4);
}
```

容量和大小：

函数原型：

+ empty() judge whether the container is empty.
+ capacity() return the capacity(容量) of container
+ size() return the length of container
+ resize(int num) modify(修改) the length of container. If modified length smaller than before the modification,then the excess elements will be deleted.
+ resize(int num,elem);If modified length bigger than before the modification, then the rest position will be filled   with 0(if you do not specify the elem,else if will be filled with elem).(elem is the second parameter)

```c++
	vector<int> v1;
	for (int i = 0;i < 10;i ++ )
	{
		v1.push_back(i);
	}
	if (v1.empty())
	{
		cout << "empty" <<endl;
	}
	else
	{
		cout << "not empty" <<endl;
		cout << "The container of v1：" << v1.capacity() << endl;
		cout << "The size of v1：" << v1.size() << endl;
	}
	v1.resize(15);
```

The insert and delete operations of vector

function:

+ push_back() tail insertion method
+ pop_back() tail deletion method
+ insert(iterator,elem) 
+ insert(iterator,num,elem) insert num elem at the specified position
+ erase(iterator) delete the specified position
+ erase(iterator,iterator)delete the elements in this compartment

get elements

v[num]	v.at(num)	

get the first/last elements

v.front()	v1.back()

Interchange container

function:

swap(vector):Swap the elements inside the two containers

If there exists a container v who is very large but has very few elements in it, this results in wasted memory. This can be solved by using the swap function, which creates an anonymous object that copies the elements of v and reallocates the capacity. We use the swap function to swap it with v. This way we reduce the size of v and the anonymous object will be cleaned up by the compiler.

```c++
vector<int> v;
	for(int i = 0;i < 100;i++)
	{
		v.push_back(i);
	}
	cout<<v.capacity()<<endl;
	cout<<v.size()<<endl;
	v.resize(3);
	cout<<v.capacity()<<endl;
	cout<<v.size()<<endl;
	vector<int> (v).swap(v);//构造函数初始化匿名对象，那么匿名对象容量将会大大减小，然后在和v交换。那么v的容量就会减小，而匿名对象的容量会变得很大，但是编译器会回收匿名对象，所以没影响
	cout<<v.capacity()<<endl;
	cout<<v.size()<<endl;
```

预留空间

功能描述：

减少vector在动态扩展时的扩展次数。

函数原型：

reverse(int len);容器预留len个元素长度，预留位置不初始化，元素不可访问。

```c++
int sum = 0;
	int* ptr = nullptr;
	vector<int> v;
	//v.reserve(100000);
	for(int i = 0;i < 100000;i++)
	{
		v.push_back(i);
		if(ptr != &v[0])
		{
			 ptr = &v[0];
			sum++;
		}
	}
	cout<<sum;
```



### queen

队列(Queue)是一种抽象数据类型，是一种先进先出(FIFO, First In First Out)的数据结构。这意味着第一个插入的数据将在第一个被删除。队列在计算机科学中的应用非常广泛，例如任务调度、广度优先搜索等。以下是队列常见的操作：

1. 主要操作：

- **enqueue**: 在队列的尾部添加一个元素。
- **dequeue**: 从队列的头部移除一个元素，并返回该元素。

2. 辅助操作：

- **isEmpty**: 判断队列是否为空。
- **isFull**: 判断队列是否已满，这通常适用于固定大小的队列。
- **front** (或 peek): 获取队列头部的元素，但不删除它。
- **size**: 返回队列中元素的个数。

```c++
#include <iostream>
#include <queue> // STL队列的头文件

using namespace std;

int main() {
    queue<int> q;

    // enqueue操作
    q.push(10); // 插入元素10
    q.push(20); // 插入元素20
    q.push(30); // 插入元素30

    // front操作
    cout << "Front element is: " << q.front() << endl; // 输出：10

    // dequeue操作
    q.pop(); // 移除元素10
    cout << "Front element is: " << q.front() << endl; // 输出：20

    // size操作
    cout << "Queue size is: " << q.size() << endl; // 输出：2

    // isEmpty操作
    cout << "Is queue empty? " << (q.empty() ? "Yes" : "No") << endl; // 输出：No

    while (!q.empty()) {
        cout << "Removing: " << q.front() << endl;
        q.pop();
    }

    cout << "Is queue empty now? " << (q.empty() ? "Yes" : "No") << endl; // 输出：Yes

    return 0;
}
```

### dequeue

double queue 双端队列

构造函数：

```c++
void print_deque(const deque<int> &de)
{
	for(deque<int>::const_iterator it = de.begin();it != de.end();it++)
	{
		cout<<*it<<"  ";
	}
	cout<<endl;
}
void test()
{
	deque<int> d;
	for(int i = 0;i < 10;i++)
	{
		d.push_back(i);
	}
	print_deque(d);
	deque<int> d1(d.begin(),d.end());
	print_deque(d1);
	deque<int> d2(10,100);
	print_deque(d2);
	deque<int> d3(d2);
	print_deque(d3);
}
```

赋值操作：

```c++
deque<int> d;
	for(int i = 0;i < 10;i++)
	{
		d.push_back(i);
	}
	//operator=
	deque<int> d1 = d;
	print_deque(d1);
	//assign
	deque<int> d2;
	d2.assign(d.begin(),d.end());
	print_deque(d2);
	deque<int> d3;
	d3.assign(10,100);
	print_deque(d3);
```



函数讲解：

1. `push_front(element)`：将元素添加到双端队列的头部。
2. `push_back(element)`：将元素添加到双端队列的尾部。
3. `pop_front()`：删除双端队列头部的元素。
4. `pop_back()`：删除双端队列尾部的元素。
5. `front()`：返回双端队列头部的元素，但不删除它。
6. `back()`：返回双端队列尾部的元素，但不删除它。
7. `empty()`：检查双端队列是否为空。
8. `size()`：返回双端队列中元素的数量。
9. `resize(num,ele = 0)`:重新规划尺寸(大小)。
10. `clear()`:清空所有元素。

```c++
#include <iostream>
#include <dequeue>
using namespace std;
int main() {
    deque<int> myDeque;

    // Push elements into the front and back of the deque
    myDeque.push_front(42);
    myDeque.push_back(10);

    // Pop elements from the front and back of the deque
    myDeque.pop_front();
    myDeque.pop_back();

    // Check if deque is empty
    if (!myDeque.empty()) {
        // Access front and back elements
        cout << "Front element: " << myDeque.front() << endl;
        cout << "Back element: " << myDeque.back() << endl;
    }

    // Get deque size
    cout << "Deque size: " << myDeque.size() << endl;

    return 0;
}
```

### stack

函数讲解：

1. `push(element)`：将元素压入栈顶。
2. `pop()`：从栈顶弹出元素。
3. `top()`：返回栈顶元素，但不将其从栈中删除。
4. `empty()`：检查栈是否为空。
5. `size()`：返回栈中元素的数量。

```c++
#include <iostream>
#include <stack>
using namespace std;
int main() 
{
    stack<int> myStack;

    // Push elements onto the stack
    myStack.push(42);
    myStack.push(10);

    // Pop element from the stack
    myStack.pop();

    // Check if stack is empty
    if (!myStack.empty()) {
        // Access top element
        cout << "Top element: " << myStack.top() << endl;
    }

    // Get stack size
    cout << "Stack size: " << myStack.size() << endl;

    return 0;
}
```

### list(链表)

函数讲解：

1. `push_front(element)`：将元素添加到链表的头部。
2. `push_back(element)`：将元素添加到链表的尾部。
3. `pop_front()`：删除链表头部的元素。
4. `pop_back()`：删除链表尾部的元素。
5. `front()`：返回链表头部的元素，但不删除它。
6. `back()`：返回链表尾部的元素，但不删除它。
7. `empty()`：检查链表是否为空。
8. `size()`：返回链表中元素的数量。

```c++
#include <iostream>
#include <list>
using namespace std;
int main() {
    list<int> myList;

    // Push elements into the front and back of the list
    myList.push_front(42);
    myList.push_back(10);

    // Pop elements from the front and back of the list
    myList.pop_front();
    myList.pop_back();

    // Check if list is empty
    if (!myList.empty()) {
        // Access front and back elements
        cout << "Front element: " << myList.front() << endl;
        cout << "Back element: " << myList.back() << endl;
    }

    // Get list size
    cout << "List size: " << myList.size() << endl;

    return 0;
}
```

### **map**























































