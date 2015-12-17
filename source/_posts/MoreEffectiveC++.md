title: More Effective C++总结
date: 2015-12-17 21:11:28
tags: C++

---

# 1.仔细区别pointers和references

Pointers和references在某些方面做着同样的事情，但两者还是有所区别：
　　没有null reference但是有nullpointers，reference总是代表着某个对象，是另一个对象的别名，reference的这一语法特定使得在使用reference之前不需要测试其有效性，在一定程度上提高了效率。在另一方面，pointers可以被重新赋值，指向另一个对象，而reference总是指向（代表）它最初获得的那个对象。
　　在实现某些操作符，例如operator[], 使用reference能够比较自然得到“被当做assignment赋值对象”的功能。

 <!--more-->

---

# 2.最好使用C++转型操作符

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
