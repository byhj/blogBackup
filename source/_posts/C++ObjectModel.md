 title: 深度探索C++对象模型总结
date: 2015-12-17 21:11:28
tags: C++

---
C++对象模型主要解释为：
- 语言中直接支持面向对象程序设计的部分
- 对于各种支持的底层实现机制

# 关于对象
　　C语言中，数据和处理数据的操作时分开声明，而C++采用抽象数据类型进行操作。在C++加上封装并未带来
布局成本，在布局以及存取时间上主要的额外负担是由virtual引起的：
- virtual function 机制：用以支持一个有效率的“执行期绑定”。
- virtual base blass : 用以实现“多次出现在继承体系中的base class, 有一个单一而被共享的实例”。
- 多重继承下的一些额外负担。

## C++对象模型
- 简单对象模型：members本身不放在objects中，object存储指向member的指针。
- 表格驱动对象模型：数据与操作分离为两个表格，object存储指向两个表格的指针。
- C++对象模型：每一个class产生一堆指向virtual functions的指针，放在virtual table(vtbl)中，每一个class
               object安插一个vptr指向相关的virtual table，每一个class关联的type_info objects信息。

## 关键词所带来的差异
当一个人感觉到比较好的时候，使用struct取代class，注意struct在C++中的逻辑意义和如何正确使用。

## 对象的差异
C++程序设计模型直接支持三种程序设计范式：
- 程序模型
- 抽象数据类型模型
- 面向对象模型
纯粹以一种范式编写程序，有助于整体行为的良好稳固。

C++以下列方式支持多态：
- 经由一组隐式的转化操作，例如把derived class指针转化为一个指向其public bass type的指针
- 经由virtual function
- 经由dynamic_cast和typeid运算符
多态的主要用途是经由一个共同的接口来影响类型的封装。

需要多少内存才能够表示一个class object:
- 其nonstatic data members的总和大小
- 任何alignment的需要而填补的空间
- 为了支持virtual 而由内部产生的任何额外负担

# 构造函数语意学

## Default Constructor的构造操作
default constructors在需要的时候被编译器产生出来，但需要注意的是这种需要时编译器方面的而不是程序的需要。
一个由于为声明constructor函数而被隐式声明的default constructor将是一个trivial constructor，在某些情况
编译器将会生成nontrivial default constructor.

- 带有Defulat Constructor 的Member Class Object
