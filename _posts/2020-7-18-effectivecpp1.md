---
layout: post
title: effective c++ notes 1, Accustoming yourself to c++
date: 2020-07-18

---
<!-- # Accustoming yourself to c++ -->

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
