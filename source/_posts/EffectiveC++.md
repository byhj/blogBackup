title: Effective C++总结
date: 2015-12-17 21:11:28
tags: C++

---

# 让自己习惯C++

## 视C++为一个语言联邦　当今的C++是个多重范型编程语言(过程形式，面向对象形式，函数形式，泛型形式，元编程形式)，我们应该将C++视为由语言组成的联邦而非单一语言,C++主要分为以下四个次语言：

- C:
C++部分继承了C的语法，这个部分没有模板，重载，继承，异常等语法。
- Object-Oriented C++:
包含classes(构造析构)， 封装，继承，多态。
- Template C++:
泛型编程的基础，带来了所谓的Template metaprogramming(TMP)编程。
- STL:
 STL是templater程序库，主要包括容器，迭代器，算法部分。

不同的次语言有着不同的规则，在编写程序时要按照你的情况进行策略的转变。
 <!--more-->

## 尽量以const, enum, inlne替代 #define
---

```
　static_cast<type>(expression)
```

## 尽量以const, enum, inlne替代 #define
　　宁可以编译器替换预处理器，因为#define不被视为语言的一部分，使用#define定义的名称没有进入符号表中，所以在追踪这类信息时会带来困扰。
预处理器盲目替换宏名称会带来更多的目标码，我们无法使用#define创建一个class专属常量，也不能提供任何封装性,下面的const提供了这些支持。

```
class GamePlayer {
  private:
  static const int NumTurns = 5;
  int scores[NumTurns];
};
```
　　如果你的编译环境不支持类内成员初始化，你可以使用所谓的“enum hack”代替，因为一个属于枚举类型的数值可以充当ints被使用。
```
class GamePlayer {
private:
   enum { NumTurns = 5};
   int socres[NumTurns];
};
```
　　在这里enum hack的行为某些方面比较像#define，因为取一个enum地址也不合法，而取一个const地址则是合法的。enum可以帮助你阻止别人
获得一个pointer或reference指向你的某个整数常量，enum hack是template metaprogramming的基础技术。

　　另一个#define误用的地方是定义像函数一样的宏，需要记住宏中所以实参都要加上小括号，这样的形式的宏很容易出现错误。我们可以使用template inline函数去替代它：

```
template<typename T>
inline void max(const T &a, const T &b)
{
  f(a > b ? a : b);
}
```
---

## 尽可能使用const
　　const允许你指定一个语义约束，而编译器会强制实施这个约束。const在使用时出现在星号左边表示被指物是常量，在右边表示指针本身是常量。
STL迭代器以指针为根据塑模处理，所以也可以用const作用于它，其功能类似。const最具威力的用法是面对函数声明时的应用：
- 在一个函数声明式内，const可以和函数返回值，各参数，函数本身产生关联
令一个函数返回常量值，可以避免客户端的赋值错误。

　　const成员函数确认了此函数可以作用于const对象，使得class接口变得容易理解。同时，使得操作const对象成为可能，以pass by reference-to-const方式传递
对象，改善了C++的程序效率。两个成员函数常量性的不同可以成为重载函数，例如可以定义两个版本的operator[], 可以针对不同的情况下进行调用。

- bitwise constness
  成员函数不更改对象内任何一个bit，这是C++对常量性的定义。
- logical constness
  一个成员函数可以修改它所出来的对象内某些bits，但只有客户端侦测不出的情况下才这样做。

  我们可以利用mutable关键字释放not-static成员变量的bitwise constness约束。
  - 在const和non-const成员函数中避免重复

```
  class TextBlock {
    public:
     const char & operator[](std::size_t pos) const
     {
       return text[pos];
     }

     char & operator[](std::size_t pos)
     {
       return text[pos];
     }
};
```

为了避免重复，我们令non-const版本调用const版本

```
class TextBlock
{
  public:
   const char & operator[](std::size_t pos) const
   {
     return text[pos];
   }
   char & operator[](std::size_t pos)
   {
     //有两次转型，第一次是为了调用const版本的[]避免递归，第二次是为了移除const
     return const_cast<char&>( static_cast<const TextBlock &)(*this)[pos] );
   }
};
```
　　需要注意的是，const成员函数承诺不改变对象的逻辑状态，所以使用const版本调用non-const版本是错误的，
这样做对象有可能因此被改动，使用了const_cast将（this*)身上的const性质解放掉。

---
## 确定对象被使用前已先被初始化
　　读取未初始化的值会导致不明确的行为，在使用对象前先初始化。对于内置类型之外的东西，其初始化责任落在构造函数上。
需要注意的是赋值和初始化的区别
```
class Test
{
public:
  //default construct + copy assignment
  Test(int a, int b)
  {
    m_a = a;
    m_b = b;
  }

private:
   int m_a;
   int m_b;
};
```
　　在这个例子是先调用了default构造函数，然后再构造函数内对成员进行了赋值操作。对象的成员变量的初始化动作发生在进入构造函数本体之前。
比较好的方式时使用成员初始化列表
```
class Test
{
public:
//copy construct
  Test(int a, int b)：m_a(a), m_b(b) {  }

private:
   int m_a;
   int m_b;
};
```
　　如果成员变量是const或references时，它们一定需要初值而不能被赋值。对于那谢赋值表现得初始化一样好的成员表里，
改用它们的赋值操作，并移动到private的某个函数，供所有构造函数调用避免重复代码。
C++成员初始化次序总是安装成员声明的顺序，所以避免两个成员变量初始化带有次序性。

- C++对于定义在不同编译单元内的non-local static 对象的初始化次序无明确的定义
non-local对象包括：
global对象，定义在namespace作用域内对象
在classes内，以及在file作用域内被声明为static的对象。

在函数内static对象是local-staic对象
```
class FileSystem {
public:
  std::size_t numDistks() const;
};
extern FileSystem fs;

class Directory {
  public:
    Directory(params)
    {
      std::size_t disks = fs.numDistks();
    }
};
//当fs在tempDir以前初始化，这才合法，但顺序不是确定的，两者都是global对象，是non-local static对象。
Directory tempDir(params);
```
　　多个编译单元内的non-local static对象经由模板隐式具现化形成。
我们可以使用设计模式中Singleton手法进行处理，C++保证函数内的local static对象会在该函数被调用期间，首次
遇上该对象之定义式时被初始化
```
class FileSystem {
public:
  std::size_t numDistks() const;
};

FileSystem & fs()
{
  staic FileSystem fs;
  return fs;
}

class Directory {
  public:
    Directory(params)
    {
      std::size_t disks = fs().numDistks();
    }
};
Directory & tempDir
{
  static Directory td;
  return td;
}
```

---
# 构造/析构/赋值运算

## 了解C++默默编写并调用哪些函数
　　一个Empty class并不空，编译器会为它声明一个copy构造函数，一个copy assignment操作符和一个析构函数。
如果你没有声明任何构造函数，编译器还会为你声明一个default构造函数，所有这些函数都是public inline。
```
class Empty {
public:
   Empty() {}
   Empty(const Empty &) {}
  ~Empty() {}

    Empty & operator = (const Empty &rhs) {}
};
```
　　在这些函数被需要(调用)时才会被编译器创建处理。这样copy构造函数和copy assignment操作符只是单纯的将来源对象的每一个non-static
成员变量拷贝到目标对象。而default构造函数和析构函数主要为编译器调用base classes和non-static成员变量的构造函数和析构函数提供环境。
当生将要成的copy assignment合法且有意义时，编译器才会执行生成操作，改变reference对象或者const成员，以及base class将copy assignment
定义为private都会导致copy assignment生成失败。


## 若不想使用编译器自动生成的函数，就该明确拒绝
　　在旧C++中，将copy构造函数或者copy assignment操作符声明为private可以阻止人们调用它们。但是member函数和friend函数还是有可能去调用
它们，这个时候会产生未定义的连接错误。我们可以将连接器的错误移至编译期，以尽早发现这类错误。具体做法如下：
```
class Uncopyable {
protected:
   Uncopyable() {}
  ~Uncopyable() {}

private:
   Uncopyable(const Uncopyable &);
   Uncopyable & operator = (const Uncopyable &);
};

class Test : private Uncopyable {

};
```
当有(member函数或friend函数)尝试拷贝Test对象的操作发生，需要产生一个copy构造函数和copy assignment函数，这些copy函数会
调用基类对应的copy函数，而基类并没有定义这样的copy函数，所以编译器会拒绝这些操作。
在C++11中可以使用 = delete函数修饰符声明成员函数为删除的，使得编译器不自动生成。

## 为多态基类声明virtual析构函数
　　当derived class对象经由一个base class指针被删除，而该base class带有一个non-virtual析构函数时，其结果未有定义，因为实际执行时对象
的derived成分没有被销毁，这造成了资源泄露。任何class只要带有virutal函数都几乎确定应该有一个virtual析构函数，如果不含virtual函数，通常表示
它并不意图被作为一个base class。
　　classes的设计目的如果不是作为base classes使用，或不是为了具备多态性就不该声明virtual析构函数。即使class完全不带virtual函数，还是可能出现
virtual析构函数问题，，当你试图去继承于STL中string,vector,list等不带virtual析构函数的class时就会出现这样的问题。
　　abstract classes不能创建对应的对象，当你希望拥有abstract classes而又没有合适的pure virtual函数时，可以将virtual函数声明为pure。这里有个要
注意的地方是：你需要为pure virtual析构函数提供一个定义，因为其派生类的析构会自动调用基类的析构函数，所以需要有一个定义。
```
class AWOV {
public:
     virtual ~AWOV() = 0;
};
//需要提供一份定义
AWOV::~AWOV(） {}
```
## 别让异常逃离析构函数
　　C++并不禁止析构函数抛出异常，当这样做往往会造成资源的泄露或者多次抛出异常导致不明确的行为。
为了避免这个问题，往往采取以下方法：
1.如果调用抛出异常时就结束程序
```
Test::~Test()
{
    try { db.close();  }
    catch(...) {
      //调用失败时记录
      std::abort()
    }
}
```
2.吞下因调用函数而发生的异常
```
Test::~Test()
{
    try { db.close();  }
    catch(...) {
      //调用失败时记录
    }
}
```
这两种方法都无法对“导致函数调用抛出异常”的情况做出反应。我们可以重新设计接口，使得客户有
机会对可能出现的问题作出反应，通过提供一个普通函数而非在析构函数中执行来对抛出的异常进行处理。
```

void Test::close()
{
  db.close();
  closed = true;
}

Test::~Test()
{
    if (!closed) {
    try {
      db.close();
    } catch(...) {
      //调用失败时记录
    }
  }
}
```

## 绝不在构造和析构过程中调用virtual函数
　　在base构造期间，virtual函数不是virtual函数，base class构造期间virtual函数不会下降到derived classed阶层，因为此时
的派生类部分还为进行构造。这个时候的对象类型是base class而不是derived class。除此之外，使用运行期类型信息(typeid或dynamic_cast)
也是同样的结果。
　　同样道理，一旦derived class析构函数开始执行，对象内的derived class成员变量就呈现未定义值，进行base class析构函数后对象就
成为一个base class对象，这个时候virtual函数并不起到想要的作用。
　　确定你的析构函数和构造函数都没有（在对象被创建和被销毁期间）调用virtual 函数，而它们调用的所有函数也都服从同一约束。
　
## 令operator = 返回一个reference to this
　　为了能够对对象进行连续赋值，赋值操作符应该返回一个reference指向操作符的左侧参数，这只是个协议并没有强制性。
```
class Test {
public:
   Test& operator = （const Test &rhs)   //同样适用于+=，-=， *=等等赋值相关运算
   {
     return *this;
   }
}

```
## 赋值对象时勿忘其每一个成分
　　当你自己定义class的copy构造函数和copy assignment操作符时，编译器不会生成将所有成员进行拷贝的默认copy操作函数。
所以如果你为class添加一个新成员变量，你必须同时修改对应的copy函数。除此之外，当你继承于这样的base class时，要注意在
派生类中的copy函数调用基类copy函数对基类成员变量进行copy操作，如果没有调用的话编译器对基类的copy可能是采取default构造函数对
成员变量进行缺省的初始化动作。

当你编写一个copy函数时，请确保：
- 复制所有local成员变量
- 调用所有base calsses内的适当的copying函数
记住不要尝试以copy构造函数去调用copy assignment，反之亦然。如果要避免代码重复的话，应该讲共同机能放进新的成员函数中，
这个函数通常为private且命名为init，然后两个copy函数共同调用这个init函数。
---

# 资源管理
当你使用了动态分配的资源，将来必须还给系统。

## 以对象管理资源
```
void test() {
  Test *pInv = createTest();
  ...   //在这里return或者抛出异常时会造成资源泄露
  delete pInv;
}
```
针对上面的资源释放的问题，我们可以使用对象自动运行析构函数机制来确保资源释放。
以对象管理资源主要有一下两个关键思想：
1.获得资源后立刻放进管理对象中,资源获取的时机便是初始化时机(RAII)。
2.管理对象运用析构函数确保资源被释放。

auto_ptr的工作原理基本与上述思想一致，auto_ptr管理的资源不允许其他auto_ptr再次指向。
因此我们可以使用引用计数型智慧指针（RCSP）shared_ptr进行资源管理，shared_ptr保存有对象指向
资源，当引用计数为0时自动删除该资源，需要注意的一点是RCSPs无法打破环状引用。

## 在资源管理类中小心copying行为
资源在构造期间获得，在析构期间释放。
当一个RAII对象被复制时:
1.禁止复制，将copying函数声明为private实现。
2.对底层资源祭出“引用计数法”, shared_ptr允许指定”删除器”来扩展使用方法。
3.复制底部资源（deep copying)，复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的copying行为。
4.转移底部资源的所有权（auto_ptr).
普遍而常见的RAII class copying行为是：抑制copying,施行引用计数法，不过其他行为也都可能被实现。

## 在资源管理类中提供对原始资源的访问
APIs往往要求访问原始资源，所以每一个RAII class应该提供一个取得其所管理之资源的方法。
在shared_ptr中get()就提供了返回指针内部原始指针的访问，在这里是进行了显示的转换。
对原始资源的访问可能经由显示转换或隐式转换，一般而言显示转换比较安全，但隐式转换对客户比较方便。


## 成对使用new和delete时要采取相同形式
如果你在new表达式使用[]，必须在相应的delete表达式中也使用[]，如果在new中不使用，那么在delete也不应该使用。
delete需要你告诉它被删除的内存有多少对象，当使用typedef对数组形式进行简化时需要注意delete的行为。
```
typedef string Address[4];
string *pstr = new Address;
delete [] pstr; //使用delete pstr是错误的
```

## 以独立语句将newed对象置入智能指针
以独立语句将newed对象存储于智能指针，如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄露。
```
processWidget(shared_ptr<Widget>(new Widget), priority());
//这里编译器对应参数的操作执行顺序没有严格的规定，当new Widget -> priority() -> shared_ptr构造
//priority抛出异常时资源泄露
//所以应该分离语句
shared_ptr<Widget> pw(new Widget);
processWidget(pw, priority);

```


---
# 设计与声明

---
# 实现

---

# 模板与泛型编程

---

# 杂项讨论

## 不要轻忽编译器的警告
```
class A{
  public:
    virtual void f() const;
};

class B : public A {
public:
  virtual void f();
};

```
这个例子中，Class B本来的意图是要重新定义A中f(),但由于B中f()为添加const标志，所以编译器提供的是
覆盖警告，这时你要确保你了解它表达的真正意图。

严肃对待编译器发出的警告信息，努力在你的编译器的最高警告级别下真去无任何警告的荣誉。
不要过度依赖编译器的报警能力，因为不同的编译器对待事情的态度并不相同。一旦依植到另一个
编译器上，你原本依赖的警告信息有可能消失。

## 让自己熟悉包括TR1以内的标准程序库
在这些里面很多特性已经成为C++11的标准
- 智能指针shared_ptr, weak_ptr
- 函数对象function
- Hash tables
- 正则表达式
- Tuples
- array
- Type traits
Boost程序库提供很多即将成为标准的组件。

## 让自己熟悉Boost
Boost主要分为以下几个组件：
- 字符串和文本处理
- 容器
- 函数对象和高级编程
- 泛型编程
- 模板元编程
- 数学和数值
- 正确性与测试
- 数据结构
- 语言间的支持
- 内存
- 杂项
