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


---

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

---

##  3, Resources Management

### 13. 以对象管理资源

获得资源后立刻放进管理对象内

管理对象运用析构函数确保资源被释放

### 14. 在资源管理类中小心copying行为

### 15. 在资源管理类中提供对原始资源的访问

提供访问原始资源（private成员）的函数

### 16. 成对使用new和delete时采用相同的形式

`new`搭配`delete`，`new`中使用`[]`， `delete`中也使用`[]`
delete加上 `[]` 后编译器会认为指针指向数组，否则它将认定指针指向单一对象
### 17. 以独立语句将newed对象置入智能指针 

考虑如下函数              
~~~
int priority();
void processWidget(std::shared_ptr<Widget> (new Widget),priority());
~~~

调用processWidget函数之后，编译器会做以下三件事
- 执行new Widget
- 调用priority
- 执行std::shared_ptr构造函数

编译器只能确定咋执行std::shared_ptr构造函数前执行new Widget

如果priority在std::shared_ptr之前调用，且抛出异常，new Widget产生的指针会丢失，导致了内存泄漏

使用单独语句将new产生的指针置入shared_ptr解决这个问题
 
 ---

## 4, Designs and Declarations


### 18. 让接口容易被正确使用，不易被误用 

好的接口很容易被正确使用，不容易被误用。努力达成这些性质(例如 explicit关键字)
“促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容
“防治误用”b包括建立新类型，限制类型上的操作，束缚对象值，以及消除用户的资源管理责任
shared_ptr支持定制deleter，需要灵活使用

### 19. 设计class犹如设计type

### 20. 宁以pass-by-renference-to-const替换pass-by-value

pass-by-value:
会调用对象内所有成员以及base class的构造/析构函数，
pass-by-renference-to-const
没有新对象被创建

slicing问题
当一个derived class对象以pass-by-value传递给一个接受base class的函数，derived class对象会仅留下一个base对象，特化性质全部被slice掉

对内置类型/STL迭代器/函数对象，使用pass-by-value往往更高效一些，其他情况下，尽量使用pass-by-renference-to-const

### 21. 必须返回对象时，别妄想返回reference
不要返回一个指向local 对象的reference，因为它在作用域结束后就被销毁
### 22. 将成员变量声明为private

### 23. 宁以non-member，non-friend替换member函数

member函数的对立面：non-member函数而不是friend函数
使用non-member，non-friend具有比friend函数更好的封装性

### 24. 若所有参数都需要类型转换，请为此采用non-member函数

考虑Rational类内重载的operator*
~~~
Rational operator*(const Rational& rhs) const;
~~~
你可以以如下形式调用operator*
~~~
Rational a, b, result;
result = a * b; 
result = a * 2; // 隐式类型转换
result = 2 * a; // error
~~~

第二条语句中编译器自动将int类型的2转换为了Rational类型
第三条语句不能通过编译，因为只有当参数位于参数列时才是隐式转换的合格参与者，*this对象不是隐式转换的合格参与者
<!-- 如果将构造函数声明为explicit以禁止隐式类型转换，下面的两条语句都不能通过编译 -->

~~~
Rational operator*(const Rational& lhs, const Rational& rhs);
~~~
将这个函数声明为non-member函数后，运算符左右两边的对象均可以被隐式转换


### 25.  考虑写一个不抛出异常的swap函数

---

## 5，Implementations

### 26. 尽可能延后变量定义式的出现时间

Ex
定义于循环外
~~~
Widget w;
for (unsigned int i = 0; i < n; ++i)
{
    w = 取决于i的某个值;
}
~~~
定义于循环内
~~~
for (unsigned int i = 0; i < n; ++i)
{
    Wiget w = 取决于i的某个值;
}
~~~

第一种写法，调用了一次ctor，n次copy assignment，一次dtor
第二种写法，调用了n次ctor和dtor

除非（1）你知道赋值成本比构造+析构成本低（2）你正在处理代码中高度敏感的部分
否则应该使用第二种写法

### 27 尽量少做转型动作


如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_cast。
如果转型是必要的，试着将它隐藏于某个函数后。客户可以随时调用该函数，而不需要将转型放入自己的代码。
使用现代C++风格的转型


### 28 避免返回handles指向对象内部成分

成员函数切勿返回自身指针/引用，会破坏封装性

### 29 为“异常安全”而努力是值得的

### 30 透彻了解inline函数的里里外外

inline应该限制在小的，频繁调用的函数上
inline只是给编译器的建议，编译器不一定执行

### 31 将文件的编译依存关系降到最低

## 6，Inheritance and Object-Oriented Design

### 32 确定你得public继承额外塑膜出is-a关系

共有继承一位is-a关系，适用于base classes上的每一件事情也一定适用于derived class上
，每一个derived class对象也是base class对象