---
layout: post
title: C++ 对象模型
date: 2020-09-27
---

总结自《深入理解C++对象模型》第一章-关于对象

### C++对象模式

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


### Reference

《深入理解C++对象模型》- Stanley B。 Lippman
