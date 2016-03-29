title: More Effective C++总结
date: 2015-12-17 21:11:28
tags: C++

---
# 基础议题

## 仔细区别pointers和references

Pointers和references在某些方面做着同样的事情，但两者还是有所区别：
　　没有null reference但是有null pointers，reference总是代表着某个对象，是另一个对象的别名，reference的这一语法特定
使得在使用reference之前不需要测试其有效性，在一定程度上提高了效率。在另一方面，pointers可以被重新赋值，指向另一个对象，
而reference总是指向（代表）它最初获得的那个对象。在实现某些操作符，例如operator[], 使用reference能够比较自然得到
“被当做assignment赋值对象”的功能。

 <!--more-->

## 最好使用C++转型操作符

　　由于旧式C语言的转型几乎可以将任何类型转换为其它类型，而没有给出每次转型更精确的意图，加上旧式转型的小括号方式在程序中可能难以辨识，所以最好才采用C++新的转型类型操作符：
```
　static_cast<type>(expression)
```
　　这是静态类型转换操作，允许基本类型之间的相互转换，但不能移除表达式的常量性，我们采用下面转换来改变表达式的常量性或易变性，这个操作由编译器执行。
```
　const_cast<type>(expression)
```
　　但要执行继承体系中“安全的向下转型或跨系转型动作”时，使用下面动态类型转换符。需要注意的是它无法应用在缺乏虚函数的类型身上，也不能改变类型的常量性。
```
　dynamic_cast<type>(expression)
```
　　最后一种转型操作几乎与编译平台相关，所以它不具备移植性，我们常用它来进行转换“函数指针”类型
```
　reinterpret_cast<type>(expression)
```

　　当你的编译平台不能支持新式转型操作时，你可以考虑采用宏去模拟这些语法：
```
 #define static_cast(TYPE, EXPR)       ((TYPE) (EXPR))
 #define const_cast(TYPE, EXPR)        ((TYPE) (EXPR))
 #define reinterpret_cast(TYPE, EXPR)  ((TYPE) (EXPR))
```

## 绝对不要以多态方式处理数组
　　多态和指针算术不能混用，数组访问时是指针算术表达式。通过base class指针删除一个由derived classes objects构成的数组，，其结果未定义。
当以数组形式传递时，编译器对数组对象的理解是声明的部分，而不是你想要的多态效果。

## 非必要不提供default constructor
　　default constructor在没有任何外来信息的情况将对象初始化，当现实世界往往需要外来信息进行操作。当你不提供default constructor时,
没有方法为数组对象指定constructor自变量，它们同样将不适用于许多template-based container classes.虽然有着这些限制，但是在一般情况下还是
不要提供default constructor。

---

# 操作符

## 对定制的“类型转换函数”保持警觉
　　C++允许编译器在不同类型之间执行隐式转换。类型转换的问题在于，在你从未打算也未预期的情况下，此类函数可能会被调用，而其结果可能是不正确
，不直观的程序行为，很难调试。存在着两种函数转换：

- 隐式类型转换操作符 ：关键词operator后加上一个类型名称
　　这类转换我们可以将operator的重载换成显式的转换专有函数，string的c_str()函数就是个很好的例子。

- 单变量constructors函数：会执行隐式的转换为类对象
　　使用explicit能够避免这一问题，编译器不能因隐式类型转换的需要调用它们，但是你还是可以通过显式类型的转换。当你无法使用explicit时，可以使用
proxy技术，将单一参数转换为一个内部类表示，这样做的话就可以执行原来的构造函数又避免类型转换动作，因为这样的转换需要两次，不合法。

## 区分increment / decrement操作符的前置和后置形式
　　通过添加一个int自变量进行后置版本的识别，编译器会为此int指定为0值，前置式返回一个reference，后置式返回返回一个const对象。
后置式返回的const能够阻止++++之类的操作，这种操作和内建类型的行为不一致，即使可以这样其行为也不是你预期那样。除此之外，后置
版本返回原来的副本也会对效率带来影响。

## 千万不要重载 && || 和 ，操作符
　　重载这些操作符会让程序从短路求值语意向函数调用转变，这两者是不同的。当函数调用动作被执行，所有参数值都必须评估完成。C++语言规范并未
明确定义函数调用动作中各参数的评估顺序。对于逗号表达式，从左往右进行评估，最后整个逗号表达式的结果以逗号右侧的值为代表。而当你重载逗号
操作符后，不能得到这些保证。

## 了解各种不同意义的new和delete
　　new operator由语言内置，不能被改变意义，总是做相同的事情：
- 它分配足够的内存，用来放置某些类型的对象。
- 调用一个constructor，为刚才分配的内存中的那个对象设定初值。
　　你能够进行改变的时用来容纳对象的那块内存的分配行为，new operator调用某个函数，执行必要的内存分配动作，你可以重写或重载那个函数，改变其行为。
这个函数称为operator new.

'''
void * operator new(size_t size);

'''
返回一个指针，指向一块原始的，未设初值的内存，当你重载这个operator new时，size_t是必须的第一参数。它只负责分配内存，不进行初始化。
你可以使用placement new对分配好的原始内存，在上面构建对象。

　　为了避免资源泄露，每一个动态分配行为都必须匹配一个相应但相反的释放动作，operator delete对于内建的delete operator关系与上面类似。
内存释放动作由函数operator delete执行，如果你只打算处理原始的，未设初值的内存，应该使用operator new和operator delete(相当于malloc和free)，
而不是new operator和delete operator.
　　数组版本的new还是那个new operator但内存以一个名为operator new[]的函数进行分配内存（array new)，delete也是同样的道理。数组版与单一版的
new operator所调用的constructor数量不同。

---

# 异常
