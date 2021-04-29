一些关于C++的notes，包括 Effective C++，Inside C++ Object Model，Modern C++ features

# Effective C++

## Accustoming yourself to c++

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

## Constructors, and Assignment Operations

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

## Resources Management

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

## Designs and Declarations


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

## Implementations

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

## Inheritance and Object-Oriented Design

### 32 确定你得public继承额外塑膜出is-a关系

共有继承一位is-a关系，适用于base classes上的每一件事情也一定适用于derived class上
，每一个derived class对象也是base class对象


总结自《深入理解C++对象模型》第一章-关于对象

# Inside C++ Object Model

## C++对象模式

在C++中如果一个class/struct 是POD（plain old data），即没有显式构造/析构/拷贝构造函数和virtual函数，那么其内存布局将会和C语言相同，元素按照声明的顺序顺序排列。当类内含virtual函数时，每一个class会产生一个virtual pointer 指向virtual table，virtual table包含了虚函数的地址（经笔者验证gcc-8.1.0中vptr被放置在类的首部）

考虑如下代码，我们有两个含虚函数的基类`B1`，`B2`，一个继承`B1`，`B2`
的子类`D`
~~~
// from microsoft docs
class B1{
public:
    virtual ~B1() {}
    void f0() {}
    virtual void f1() {}
    int int_in_b1;
};
class B2{
public:
    virtual ~B2() {}
    virtual void f2() {}
    int int_in_b2;
};
class D : public B1, public B2{
public:
    void d() {}
    void f2() override {}
    int int_in_d;
};

int main()
{
    D *d = new D();
    std::cout << d << "\n"
              << &d->int_in_b1 << "\n"
              << &d->int_in_b2 << "\n"
              << &d->int_in_d << "\n";
    std::cout << sizeof(*d);
}
~~~
gcc 8.1.0 x86_64-w64-mingw32下的测试结果，我们可以通过测试结果推断D的内存布局
~~~
0x721320   // 0 : vptr for b1 table
0x721328   // 8 : int_in_b1
           // 12: 4 padding bytes
           // 16:vptr for b2 table
0x721338   // 24:int_in_b2
0x72133c   // 28_int_ind
32
~~~

### 指针的类型

指针的值是其所指向对象的地址，指针的类型告诉了编译器解释如何解释某个地址中的内存。
如果我们有一个基类`A`，一个继承自`A`的子类`B`，我们通过如下方式声明两个指针。
~~~
B b;
A *pa = &b;
B *pb = &b;
~~~
指针`pa`，`pb`都指向`b`的首地址，它们的差别是，`pa`只覆盖了大小为`sizeof(A)`的区域。消除此差别的方式是通过类型转换的方式将`pa`转换成`B*`，即改变编译器解释地址的方式。


<!-- 
virtual class 中会有一个virtual pointer 指向virtual table作为class的第一个元素，virtual table包含了虚函数的地址

gcc-8.1.0中vptr被放置在类的首部

在implemention中，basic class

usual implementations, basic does not get derived's vptr; *basic's vptr will point to derived's vtable

The virtual method table is the same for all objects belonging to the same class -->
<!-- 虚函数作用
support dynamic dispatch 
or run-time method binding -->

<!-- binary tree dispatch

Whenever virtual function is called using base class reference or pointer it cannot be inlined (because call is resolved at runtime), but whenever called using the object (without reference or pointer) of that class, can be inlined because compiler knows the exact class of the object at compile time. -->

# Modern C++ features

总结自

https://isocpp.org/

https://changkun.de/modern-cpp/zh-cn

# *C++ 11*

## General Features

### auto

auto关键字用于在变量初始化时推断变量的类型

```
// example

// in c++98
template<class T> void printall(const vector<T>& v)
{
    for (typename vector<T>::const_iterator p = v.begin(); p!=v.end(); ++p)
        cout << *p << "\n";
}
// in c++11
template<class T> void printall(const vector<T>& v)
{
    for (auto p = v.begin(); p!=v.end(); ++p)
        cout << *p << "\n";
}
```

### decltype

decltype(E) 代表表达式 E 的类型

```
// example

void f(const vector<int>& a, vector<float>& b)
{
    typedef decltype(a[0]*b[0]) Tmp;
    for (int i=0; i<b.size(); ++i) {
        Tmp* p = new Tmp(a[i]*b[i]);
        // ...
    }
    // ...
}
```

> Note: Prefer just using auto when you just need the type for a variable that you are about to initialize. You really need decltype if you need a type for something that is not a variable, such as a return type.

### Range-for statement

用range-based for loop和auto替代原生C中的for语句

```
// example

void f(vector<double>& v)
{
    for (auto x : v) cout << x << '\n';
    for (auto& x : v) ++x;  // using a reference to allow us to change  the value
}

//x.begin() and x.end stand for the first and last element of container.
```

### Initializer lists

除非你打算和C++98编译器共享你的代码，否则应使用initializer_list重载构造函数

### Uniform initialization syntax and semantics

C++11中允许用initializer_list做一切初始化工作

```
// example 

X x1 = X{1,2}; 
X x2 = {1,2};   // the = is optional
X x3{1,2}; 
X* p = new X{1,2}; 
struct D : X {
    D(int x, int y) :X{x,y} { /* ... */ };
};
struct S {
    int a[3];
    S(int x, int y, int z) :a{x,y,z} { /* ... */ }; // solution to old problem
};
```



### lambdas

lambda表达式/匿名函数

### noexcept to prevent exception propagation

在noexcept函数中抛出异常会调用std::terminate()终止程序，为编译器的优化提供了更大的空间

```
//example

constexpr int len_2_constexpr = 1 + 2 + 3;
char arr_4[len_2_constexpr];         // 合法
```

### constexpr

C++11 提供了 constexpr 让用户显式的声明函数或对象构造函数在编译期会成为常量表达式，如果编译器能够在编译时就把这些表达式直接优化并植入到程序运行时，将能增加程序的性能

### nullptr – a null pointer literal 

nullptr替代NULL避免歧义

### inline namespaces

内联命名空间声明之后，就可以在外层命名空间不适用前缀而直接使用它们了

see: https://blog.csdn.net/craftsman1970/article/details/82872497

### Rvalue references and move semantics

左值是表达式（不一定是赋值表达式）后依然存在的持久对象。
右值是指表达式结束后就不再存在的临时对象。

**右值引用，移动语义**

考虑C++98中如下情况

```
std::vector<int> foo() {
    std::vector<int> temp = {1, 2, 3, 4};
    return temp;
}

std::vector<int> v = foo();
```

调用foo()后，调用拷贝构造函数将返回值temp赋给v，再调用temp的析构函数，造成了不必要的开销

在 C++11 之后，编译器为我们做了一些工作，此处的左值 temp 会被进行此隐式右值转换， 等价于 static_cast<std::vector<int> &&>(temp)，进而此处的 v 会将 foo 局部返回的值进行移动，被 vector 的**移动构造函数**引用，从而延长生命周期，并将这个右值中的指针拿到，保存到了 obj 中，而将亡值的指针被设置为 nullptr，防止了这块内存区域被销毁。

从而避免了无意义的拷贝构造，加强了性能。

C++11 提供了 std::move 这个方法将左值参数无条件的转换为右值

## Class Features


### default,delete关键字

default：使用默认构造/析构/拷贝构造函数
delete：禁用构造/析构/拷贝构造函数

```
// example : default

class X1 {
    // ...
    X1& operator=(const X1&) = default;    // default copying
    X1(const X1&) = default;
};
// example : delete

class X2 {
    // ...
    X2& operator=(const X2&) = delete;    // Disallow copying
    X2(const X2&) = delete;
};

```

### 移动拷贝，移动构造


接受右值时调用移动拷贝，移动构造函数
接受左值时调用拷贝构造函数，拷贝构造运算符

```
class X1 {
    // ...
    X1& operator=(X1&&); // move assignment
}

class X2 {
    // ...
    X2& operator=(X2&&); // move constructor
}
```


### 委托构造

构造函数可以以初始化列表的方式调用其他构造函数

```
// example

class X {
    int a;
public:
    X(int x) { if (0<x && x<=max) a=x; else throw bad_X(x); }
    X() :X{42} { }
    X(string s) :X{lexical_cast<int>(s)} { }
    // ...
};
```

### 继承构造

### override关键字

### final关键字


再类/函数后加final关键字禁止重载

### Explicit conversion operators

```
// example

struct S { S(int) { } };

struct SS {
    int m;
    SS(int x) :m(x) { }
    explicit operator S() { return S(m); }  // because S don't have S(SS)
};

SS ss(1);
S s1 = ss;  // error; like an explicit constructor
S s2(ss);   // ok ; like an explicit constructor
void f(S); 
f(ss);      // error; like an explicit constructor

```

### In-class member initializers

C++98只允许static const int类型变量类内初始化
C++11允许所有成员以初始化列表的方式进行初始化


## Other Types

### enum class

enum的问题
- Conventional enums implicitly convert to an integer, causing errors when someone does not want an enumeration to act as an integer.
- Conventional enums export their enumerators to the surrounding scope, causing name clashes.
- The underlying type of an enum cannot be specified, causing confusion, compatibility problems, and makes forward declaration impossible.


### long long – a longer integer

64位int

<!-- ## templates
### Template aliases
### Variadic templates -->

## Misc

### __cpluscplus

c++11后```__cplusplus```代表 *201103L*

### Preventing narrowing

C++11中用```{}```初始化可以避免narrow的问题

```
// example: C++98

int x = 7.3;        // Ouch!
void f(int);
f(7.3);         // Ouch!
```

```
// example: C++11

int x0 {7.3};   // error: narrowing
int x1 = {7.3}; // error: narrowing
double d = 7;
int x2{d};      // error: narrowing (double to int)
char x3{7};     // ok: even though 7 is an int, this is not narrowing
vector<int> vi = { 1, 2.3, 4, 5.6 };    // error: double to int narrowing
```
### Right-angle brackets

解决了右尖括号编译问题
```
list<vector<string>> lvs;
```

### static_assert

编译期的```assert```，不会造成任何运行期性能损失

### Raw string literals

### Alignment

字节对齐，不同编译器的方式不同
```
struct alignas(8) X
{
    char a;
    int  b;
    double c;
};
```

## Libiary

### 智能指针

std::shared_ptr
std::unique_ptr
std::weak_ptr

### std::array
替代原生数组
### function & bind
### type traits
### Garbage collection ABI 
### tuple 
### Random number generation 
### Scoped allocators
--- 
