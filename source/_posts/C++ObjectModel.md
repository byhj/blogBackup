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

<!--more-->

## C++对象模型
- 简单对象模型：members本身不放在objects中，object存储指向member的指针。
- 表格驱动对象模型：数据与操作分离为两个表格，object存储指向两个表格的指针。
- C++对象模型：每一个class产生一堆指向virtual functions的指针，放在virtual table(vtbl)中，每一个class
               object安插一个vptr指向相关的virtual table，每一个class关联的type_info objects信息。
               vptr的设定和重置都由每一个class的constructor, destructor, copy assignment自动完成。

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

---

# 构造函数语意学

## Default Constructor的构造操作
default constructors在需要的时候被编译器产生出来，但需要注意的是这种需要时编译器方面的而不是程序的需要。
一个由于为声明constructor函数而被隐式声明的default constructor将是一个trivial（无用） constructor，在某些情况
编译器将会生成nontrivial default constructor.

- 带有Defulat Constructor 的Member Class Object：
  　　在constructor真正被调用生成，编译器在包含这样member的constructor中自动扩张，将member的constructor行为先进行
  运行，其调用顺序根据member objects在class中的声明顺序决定。

- 带有Default Constructor 的Base Class
 　与上面同理，bass class的constructor先进行调用。

- 带有一个Virtual Function的Class
　　class声明或者继承一个virtual Function  /  class派生自一个继承链，其中有一个或多个virtual base classes.
一个virtual function table 会被编译器生成，内放virtual function地址。vptr内含vtbl的地址。

- 带有Virtual Base Class 的 Class

## Copy Constructor的构造操作
有三种情况使得一个object的内容作为另一个class object的初值：
- 对一个object做显示的初始化操作
- 当object被当做参数交给某个函数时
- 当函数传回一个class object时

Default Memberwise Initialization:为定义一个explicit copy constructor时，将object使用递归方式拷贝每个成员。

当class不展现Bitwise Copy Semantics时会合成copy constructor:
- 当class内含一个声明有copy constructorh的member object
- 继承的base class有一个copy constructor(不论是显示声明还是合成而得)
- 当class声明一个多多个virtual functions
- 当class派生于继承链，其中有一个或多个virtual base classes

## 程序转化语意学

## 成员们的初始化队伍
- 初始化一个reference member
- 初始化一个const member
- 调用一个base class的constructor,而他拥有一组参数
- 调用一个member class的constructor， 而它有一组参数
initialization list的初始化会被以适当的顺序在constructor之内安插初始化操作，并且在任何explicit user code之前。

---

# Data 语意学
- 语言本身所造成的额外负担
- 编译器对于特殊情况所提供的优化处理
- Alignment限制

## Data Member的绑定
   一个inline函数实体，在整个class声明未被完全看见之前，是不会被评估求值得。

## Data Member的布局
　　static data merbers不会被放进class的布局中，members的顺序根据编译器而定，较晚出现的members有较高的地址。

## Data Member的存取
- Static Data Members
　　static 成员只有一个实例，通过一个指针和对象存取member结果一样。若取一个static data member的地址会得到一个指向
其数据类型的指针。name-mangling对其进行独一性的编码。

- Nonstatic Data Members
　　在成员函数内，data members通过隐式的this指针进行访问，不同data数据的访问是通过对class object地址进行偏移进行访问。
Data members的地址在编译期即可获得，当使用继承之类的机制，data members的存取效率会受到影响。

## 继承与Data Member
　　具体继承并不会增加空间或者存取时间的额外负担。但加上多态机制之后情况就会有所不同，vtbl 和vptr的创建，constructor和destructor对
vtbl和vptr的设定和销毁都会带来空间和时间上的负担。
　　将vptr放在class object的尾端可以保留base class C struct的对象布局。将vptr放在class object的前端对于在多重继承下，通过指向class
members的指针调用virtual function带来一些帮助。

## 多重继承


## 虚继承

## 对象成员的效率

## 指向Data Members的指针
 　　取一个nonstatic data member的地址，将会得到它在class中的offset
---

# Function 语意学

## Member的各种调用方式

### Nonstatic Member functions
- 改写函数的signature以安插一个额外参数（this)到member function以提供存取管道，使得class object得以将此函数调用。
- 将每一个“对nonstatic data member”的存取操作改为经由this指针来存取。
- 将member function通过mangling重新写成一个独一无二的·外部函数。

### Virtual Member functions
  通过虚运行机制进行调用。
### Static Member functions
- 没有this指针
- 不能直接存取class中的nonstatic members
- 不能够被声明为const, volatile， virtual
- 不需要经由class object才被调用
- 取一个这样的地址将获得其内存中的地址吗,是一个nonmember函数指针。

## Virtual Member functions
　　多态以一个public base class的指针或引用寻址出一个derived class object。每个多态class object增加两个members
一个字符串或数据表示class的类型和一个指向某个表格的指针（vptr),为了找到函数地址，每一个virtual function被指派一个表格
索引值。

### 多重继承下得Virtual functions

## 函数的效能

## 指向Member Function的指针
　　取一个nonstatic member function的地址得到的时它在内存中真正的地址，这个值不是完全的，需要被绑定到某个class object上才能
够通过它调用该函数。

## Inline Functions
- 分析函数定义
- 真正的inline函数扩展操作是在调用的那一点上。

---

# 构造，析构，拷贝语意学
纯虚函数的存在：可以定义和静态调用一个pure virtual Function, pure virtual function一定需要进行定义，派生类需要进行相关调用。

## "无继承"情况下得对象构造
Plain OI Data  ：struct
抽象数据类型    ：ADT（Class)
带virtual function类 ：constructor需要再任何base class construtors调用之后，在任何使用者供应的代码之前进行附加vptr初始化。
这个时候，copy constructor和copy assignment operator不再是trivial,同样需要对vptr进行操作。

## 继承体系下得对象构造
- 虚继承：解决virtual base class的共享性问题

## vptr初始化语意学
- 当一个完整的对象被构造起来时
- 当一个subobject constructor调用了一个virtual function

## 对象复制语意

## 对象的效能

## 析构语意学
- 如果object内含一个vptrm那么首先重设相关的virtual table
- destructor的函数本体现在被执行
- class拥有member class objects,而后者有destructors，那么它们会以声明顺序的相反顺序被调用
-

---

# 执行期语意学

## 对象的构造和析构
将object尽可能放置在使用它的那个程序区段附近，这样做可以节省非必要的对象产生操作和摧毁操作。
- 全局对象: 需要静态初始化
- 局部静态对象: 初始化一次
- 对象数组：

## new和delete运算符
- 通过设当的new运算符函数实例，配置所需的内存
- 将配置得来的对象设立初值
