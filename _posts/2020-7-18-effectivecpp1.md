---
layout: post
title: effective c++ notes
date: 2020-07-18

---
<!-- # Accustoming yourself to c++ -->

##  1, Accustoming yourself to c++

### 1. 视C++为一个语言联邦

C++经过数十年的发展，已经难以用单一语言描述，而是从四个次语言组成的语言联邦
这个联邦包括：C风格的C++，Object Oriented C++，Template C++，STL

### 2. 用 const，enum，inilne，代替 \#define

\#define 是预处理器指令，在定义常量时，在编译期之前就被预处理器移走，不被编译器看到，声明单纯的常量尽量用const 常量代替

~~~
const int num = 10;
~~~

类内常量的创建方式

- static const 常量
~~~
class A{
    static const int num;
    int arr[num];
}
~~~

- enum hack
~~~
class A{
    enum {num = 5};
    int arr[num];
}
~~~

enum hack优点：
更像\#define，不会导致不必要的内存反派
是一个常量，无reference和pointer



> \#define无作用域概念，不能在类内使用

template inline函数兼顾安全性和效率，可用于替代形似函数的宏（macro）


### 3. 尽量使用 const

const修饰指针时判断常量的方式
const出现在星号左边：被指物是常量
~~~
const int* p = nullptr;
int const* p = nullptr;
~~~
const出现在星号右边：指针自身是常量
~~~
int* const p = nullptr;
~~~

### 4. 确定对象使用前已先被初始化

永远在对象使用前将它初始化
确保构造函数将对象的每一个成员都初始化

推荐使用member initialization（成员初值列）替换赋值动作
在构造函数中做赋值动作首先调用default构造函数设初值，再对其赋值
初值列的参数直接被用于构造函数的实参，效率更高

c++有固定的成员初始化次序，初值列列出的成员变量次序应与class声明次序相同

C++对不同编译单元的non-local static 对象初始化次序无明确定义
请以local static 对象替代non-local static 对象

##  2, Constructors, and Assignment Operations


### 5. 了解C++默认编写并调用那些函数

一个如下所示的空类

~~~
class Empty {};
~~~

经C++处理过后，编译器会自动为其添加默认的构造函数，析构函数，拷贝构造函数和copy assignment运算符，且都是public的，如果人为的定义了某个函数，编译器就不会提供默认的函数，在默认构造函数中，编译器会调用base class 和 non-static成员变量的构造函数

C++11可以通过default关键字强制使用编译器提供的函数

如果base class将copy assignment运算符声明为private，编译器会拒绝为derived class生成默认copy assignment运算符

<!-- Ex：
~~~
class A {
const 
};
~~~ -->

如果class内有reference/pointer/const成员变量，使用默认的copy assignment运算符时编译器会拒绝编译，必须自己定义copy assignment运算符，


### 6. 若不想使用编译器自动生成的函数，应该明确拒绝

将构造函数，析构函数，拷贝构造函数和copy assignment运算符声明为private且不定义以禁止调用

C++11方法：使用delete关键字禁用类和函数

将class继承自uncopyable class以禁止拷贝构造

### 7. 为多态基类声明virtual析构函数

多态性质的base class应该声明一个virtual析构函数，
如果设计的class没有多态性，就不该声明virtual析构函数函数

C++11可以使用final关键字禁止类和成员函数的派生

### 8. 别让异常逃离析构函数


析构函数不要吐出异常，会引发内存泄漏

### 9.  绝不再构造和析构函数中调用virtual函数

在构造函数期间调用的virtual函数不下降至derived class层，因为base class的构造函数会先调用，析构函数同理

### 10. 令operator= 返回 reference to *this

允许连锁赋值

### 11. 在operator= 中处理自我赋值

用证同测试检验自我赋值

### 12. 复制对象时勿忘其每一个成分

在derived class的copy assignment和拷贝构造函数中调用base class的相应函数

在copy assignment运算符中调用拷贝构造函数和在拷贝构造函数中调用copy assignment运算符都是不可取的

<!-- ##  3, Resources Management

### 13. 以对象管理资源

### 14. 在资源管理类中小心copying行为

### 15. 在资源管理类中提供对原始资源的访问

### 16. 成对使用new和delete时

### 17. 以独立语句将newed对象置入智能指针 -->


<!-- 
## 4, Designs and Declarations


### 18. 
### 19. 
### 20. 
### 21. 
### 22. 
### 23. 
### 24. 
### 25.  -->