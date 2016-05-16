title: More Effective C++总结
date: 2015-12-17 21:11:28
tags:
 - C++

---
# 基础议题

## 仔细区别pointers和references

Pointers和references在某些方面做着同样的事情，但两者还是有所区别：
　　没有null reference但是有null pointers，reference总是代表着某个对象，是另一个对象的别名，reference的这一语法特定使得在使用reference之前不需要测试其有效性，在一定程度上提高了效率。在另一方面，pointers可以被重新赋值，指向另一个对象，而reference总是指向（代表）它最初获得的那个对象。在实现某些操作符，例如operator[], 使用reference能够比较自然得到“被当做assignment赋值对象”的功能。

## 最好使用C++转型操作符

　　由于旧式C语言的转型几乎可以将任何类型转换为其它类型，而没有给出每次转型更精确的意图，加上旧式转型的小括号方式在程序中可能难以辨识，所以最好才采用C++新的转型类型操作符：
 <!--more-->
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

## 利用destructors避免内存泄露
　　以一个对象存放“必须自动释放的资源”并依赖该对象的destructor释放，可以在exceptions出现时避免资源泄露。

## 在constructors内阻止资源泄露
　　exception的发生可能由于operator new无法分配足够内存时，这时控制权被移出constructor之外，而析构函数对已构造完成的对象才进行
释放操作。这个时候，我们需要将所有可能的exceptioons捕捉起来，执行某些清理工作，然后从新抛出exception使它继续传播出去。当member initialization\
lists时，把try catch放在private member functions内，然后进行处理。

## 禁止异常流出destructors之外
　　destructor在两种情况下会被调用：
- 当对象在正常状态下被销毁，也就是当它离开了它的生存空间或是被明确地删除
- 当对象在exception处理机制的statck-unwinding(展开)机制-销毁
可以避免terminate函数在exception传播过程的stack-unwinding， 可以协助确保destructors完成其应该完成的所有的事情。


## 了解“抛出一个exception”与”传递一个参数”或“调用一个虚函数”之间的差异
　　函数参数和exceptions的传递方式都有三种，by value, vy reference, by pointer。但是你调用一个函数，控制权最终会回到调用端。
而当你抛出一个exception,控制权不会再回到抛出端。不论exception是以哪种方式参数传递，都会发生复制行为，catch得到是副本，这个复制行为
是由对象的copy constructor执行的，该复制动作永远是以对象的静态类型为本。
　　exceptions与catch子句相匹配过程中，仅有两种转换可以发生：
- 继承架构中的类转换
- 从一个“有型指针”转为”无型指针”

## 以by reference方式捕捉exceptions
　　catch by reference可以避免对象删除问题。

## 明智运用exception specifications

## 了解异常处理的成本
- 必须付出一些空间，放置某些数据结构（记录着那些对象已被完全构造）
- try语句块

---

# 效率

## 谨记80-20法则
　　软件的整体性能几乎总是有其构成要素的一小部分决定。借助程序分析器查找出瓶颈在哪一部分，然后再进行优化。

## 考虑使用lazy evaluation(缓式评估)
- Reference Counting (引用计数) ：在你真正需要之前，不必着急为某物做一个副本，可以采用拖延政策直到需要修改时才提供副本操作。
- 区分读和写 ： operator[]的读写区分
- Lazy Fetching（缓式取出）：避免非必要的数据库读取动作
- Lazy Expression Evaluation(表达式缓评估) ：避免不必要的数值计算动作

## 分期摊还预期的计算成本。
   超急评估：令他超前进度地做“要求以外”的更多工作。
   如果你预期程序常常会用到某个计算，你可以降低每次计算的平均成本，办法就是设计一份数据结构以便能够极有效率地处理需求。
  Catching和prefetching(预先取出)都是其中的案例。
  但是较佳往往导致较大的内存成本。

## 了解临时对象的来源
　　C++中真正的临时对象时不可见的，由编译器隐式调用生成。一般而言，函数调用时隐式转换和函数返回对象时会生成这类对象。
你可以声明reference-to-const避免这类问题。

## 协助完成返回值优化（RVO)
　　编译器在返回对象时，可以执行返回值优化。
'''
const TestClass operator+(const TestClass &lhs, const TestClass &rhs) {
  return TestClass(lhs.val + rhs.val);
}
'''

## 利用重载技术避免隐式类型转换
　　每个重载操作符必须获得至少一个用户定制的自变量，我们可以通过提供多种参数的重载函数避免临时对象的生成。

## 考虑以操作符复合形式取代其独身形式
　　我们一般使用复合形式来实现operator的单一版本，单一版本需要返回一个新对象，而复合版本则是直接将结果写入
其左端自变量，不需要产生一个临时对象来放置返回值。

## 考虑使用其他程序库
　　C++的IO操作比C的慢很多，可以视情况选择合适的程序库。

## 了解virtual functions, multiple inheritance, virtual base class, runtime type identification的成本
　　vtbl和vptr需要额外的内存空间，换页动作由此受到影响。虚函数本身并不构成性能上的瓶颈，真正运行时期成本发生在inlining互动的时候。
多重继承往往导致virtual base classes虚拟基类的需要，这可能导致对象内的隐式指针增加。RTTI让我们在运行时期获得objects和classes的
相关信息，需要空间存储type_info信息，使用typeud去获得class的对应type_info，type_info放在vtbl中。

---

# 技术
## 将constructor 和 non-member functions虚化
　　virtual constructo是某种函数，视其获得的输入，可产生不同类型对象。从磁盘或者磁带读取对象信息经常被用到。

virtual copy constructor返回一个指针，指向其调用者的一个新副本，通常以clone()命名（Prototypee）。
当derived class重新定义其base class的一个虚函数时，不再需要一定得声明与原来相同的返回类型。
'''
class Base {
  virtual Base * clone() const = 0;
};

class A : public Base {
  virtual A * clone() const {
    return new A(*this);
  }
};

class B : public Base {
  virtual B * clone() const {
    return new B(*this);
  }
};

'''

non-member functions的虚化：写一个虚函数做实际工作，再写一个什么都不做的非虚函数，只负责调用函数。

'''
class Base {
  virtual ostream & print(ostream &s) const = 0;
};

class A : public Base {
  virtual ostream & print(ostream &s) const;
};

class B : public Base {
  virtual ostream & print(ostream &s) const;
};
inline ostream &operator << (ostream &s, const Base &b) {
  return b.print(s);
}

'''

## 限制某个class所能产生的对象数量
- 允许零个或一个对象
阻止对象生成，只要将constructors声明为private，而利用单例模式static操作则可以生成一个对象。

- 不同的对象构造状态

- 允许对象生生灭灭

- 一个用来计算对象个数的base class

## 要求或禁止对象产生于heap之中
- 要求对象产生于heap之中（head-based objects)
让destrutor成为private,而constructors为public,导入一个伪destructor函数调用真正的private。

- 判断某个对象是否位于heap中

- 禁止对象产生于heap之中
对象被直接实例化，对象被实例化为derived class objects内的base classes成分，对象被内嵌于其他对象之中。


## 智能指针


## proxy classes代理类
　　用来代表其他对象的对象，称为proxy objects（替身对象）,而用以表现proxy objects者，我们称为proxy classes。

---

# 杂项讨论

## 在未来时态下发展程序

　　好的软件对于变化有良好的适应能力。
- 以C++本身(而非只是注释或说明文件)来表现各种规范
- 避免"demand-paged"式的虚函数
- 为每一个class处理assignment和copy construction动作
- 努力让classes的操作符和函数拥有自然的语法和直观的语义
- 让classes容易被正确使用，不容易被误用
- 努力写出可移植代码
- 设计你的代码，使：“系统改变所带来的冲击”得以局部化
- 提供完整的classes,即使某些部分目前用不到
- 设计你的接口，使有利于共同的操作行为，阻止共同的错误
- 尽量使你的代码一般化，除非有不良的巨大后果

## 将非尾端类（non-leaf classes）设计为抽象类

　　让原来的继承base类成为子类，添加一个抽象基类，这样做可达到允许同型赋值，部分赋值和异型赋值都被禁止的功能。
我们可以将其中的virtual destrutor函数声明为纯虚函数来避免提供一个不适合的纯虚接口，这个纯虚析构函数必须被定义，
因为derived classes在析构时会调用base classes的析构函数。
　　抽象基类降低了“企图以多态方式对待数组”的机会，让operator=的行为更容易被了解。强迫你明白认知有用之抽象性质的
存在。只有在原有的具体类被当做基类使用，才强迫导入一个新的抽象类。
 　　当你需要扩展一个程序库而又不能修改代码时，只有少数可行的选择：
 - 将你的具体类派生自既存的具体类
 - 试着在程序库的继承体系中找到一个更高层的抽象类
 - 以“你所希望继承的那个程序库类”来实现自己的新类，将程序库类对象作为新类的member
 - 写一些non-member functions以供应你希望加入class内部去的机能

 ## 如何在同一个程序中结合C++和C

- Name Mangling（名称重整）
　　Name Mangling是一种程序，通过它C++编译器为程序内的每一个函数编出独一无二的名称，在C中没有重载所以不需要此操作。
要抑制Name Mangling需要使用C++的extern “C”的命令，它是一个叙述，说明修饰的部分应该以C语言的方式调用。如果需要同时
处理C和C++的使用，可以采用预处理器符号__cplusplus, 这个符号只针对C++才用定义.

'''
//如果在C++环境中我们使用C编译方式
#ifdef __cplusplus
extern "C" {
#endif

// code

#ifdef __cplusplus
}
#endif

'''

- Statics的初始化
　　stait class对象，全局对象，namespace内的对象以及文件范围内的对象，其constructor总在main以前得到执行。
而static initialization产生对象的destructors在main调用后。为了使得main是程序起点，很多编译器在main开始出安插
一个函数调用的特殊函数完成static initialization.
　　尽量使用C++编写main函数，可以调用的命名mian函数进行实际的操作,这样可以保证static的构造和析构被合理调用。
'''
extern "C"
int realMain(int argc, char *argv[]);

int main(int argc, char *argv[]) {
  return realMain(argc, argv);
}
'''

- 动态分配内存
　　C++使用new和delete， C使用malloc和free进行内存分配，char *strdup（const char *ps)这个函数的内存分配就会
根据不同的调用采取不同的内存分配方式，所以尽量避免调用标准程序库外的函数。

- 数据结构的兼容性
　　在C和C++之间对数据结构做双向交流，应该是安全的，前提是那些结构的定义式在C和C++中都可编译。

## 让自己习惯于标准C++语言
　　
