title: Effective C++总结
date: 2015-12-17 21:11:28
tags: C++

---

# 让自己习惯C++

## 视C++为一个语言联邦　
当今的C++是个多重范型编程语言(过程形式，面向对象形式，函数形式，泛型形式，元编程形式)，
我们应该将C++视为由语言组成的联邦而非单一语言,C++主要分为以下四个次语言：
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
需要注意的是赋值和初始化的区别.
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
　　在这个例子是先调用了default构造函数，然后再构造函数内对成员进行了赋值操作。
对象的成员变量的初始化动作发生在进入构造函数本体之前。比较好的方式时使用成员初始化列表。
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

## 让接口容易被正确使用，不易被误用
　　如果客户企图使用某个接口而却没有获得他所预期的行为，这个代码就不应该通过编译，设计良好的接口必须考虑客户可能做出什么样的错误。
许多客户端的错误可以通过导入新类型而得到解决。让types容易被正确使用，不容易被误用，尽量使得你的types行为与内置types一致。
阻止误用的办法包括建立新类型，限制类型上的操作，束缚对象值，消除客户的资源管理责任。
shared_ptr支持定制型删除器，可用来防范DLL问题：对象在动态链接程序库被new创建，却在另一个DLL内被delete。

## 设计class犹如设计type
当你定义一个新class，也就定义了一个新type，好的types有自然的语法，直观的语义，以及一或多个高效实现品。
- 新type的对象应该如何被创建和销毁
- 对象的初始化和对象的赋值该有什么样的区别
- 什么是新type的合法值
- 你的新type需要配合某个继承图系
- 你的新type需要什么样的转换
- 什么样的操作符合函数对此新type而言是合理的
- 什么样的标准函数应该驳回
- 什么是新type的未声明接口
- 你的新type有多么一般化

## 宁可pass-by-reference-to-const替换pass-by-value
　　pass-by-value需要创建副本，带来的事对象的构造成本，在函数传递时实参会调用对象的copy构造函数进行复制。而采用pass-by-reference-to-const
则可以避免构造函数或析构函数的调用，除此之外by reference方式传递参数也可以避免对象切割问题（当derived class对象以by value方式传递到一个base
class对象上，derived 部分会被抛弃）。
　　以上规则并不适用于内置类型，STL的迭代器和函数对象，对他们来说，pass-by-value往往比较适合。

## 必须返回对象时，别妄想返回其reference
　　绝不要返回pointer或reference指向某个local stack对象，因为函数调用结束后local对象会被销毁。
也不要返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static对象而有
可能同时需要多个这样的对象。

## 将成员变量声明为private
将成员变量声明为private，可赋予客户范文数据的一致性，可细微划分访问控制，许诺约束条件获得保证，并提高class作者以充分的实现弹性。
protected并比public更具封装性，封装性与其内容改变时可能造成的代码破坏量成反比。

## 宁以non-member,non-friend替换member函数
宁以non-member,non-friend替换member函数，这样可以增加封装性，包裹弹性和机能扩充性。

## 若所有参数皆需类型转换，请为此采用non-member函数

## 考虑写出一个不抛出异常的swap函数

---
# 实现

## 尽可能延后变量定义式的出现时间
因为异常的抛出和程序错误可能会使早些定义好的变量没有析构，造成资源泄露。

## 尽量少做转型动作
const_cast<T>();     //常量性移除
dynamic_cast<T>();   //安全向下转型
reinterpret_cast<T>(); //低级转型
static_cast<T>();    //隐式转型

## 避免返回handles指向对象内部成分
避免dangling handles

## 为“异常安全”而努力是值得的
不泄露任何资源
不允许数据败坏

## 彻底了解inlining的里里外外
　　对函数调用使用函数本体进行替换，增加目标码，导致额外的换页行为，降低指令高速缓存装置的击中率。
inline只是对编译器的申请，而不是强制命令，一般放在头文件中。编译器会拒绝过于复杂（带有循环或递归）
的函数inline，一个函数是否真正inline，取决于你的建置环境，主要取决于编译器。
编译器不对通过函数指针而进行的调用实施inlining,当程序需要取某个inline函数的地址时也不会inline.


## 将文件间的编译依存关系降至最低
　　使用object references或object pointer完成任务，不用objects.
尽量以class声明式替换class定义式。

1.pimpl idiom
2.interface class
---
# 继承与面向对象设计

## 确定你的public继承塑模出is-a关系
　　能够施加于base class对象身上的每件事情，都可以施加于derived class对象上；每一个derived class也都是一个base class.
企鹅与鸟，矩形和正方形都是错误的public继承。

## 避免遮掩继承而来的名称
　　derived class作用域被嵌套在base class作用域内，所以derived classes内的名称会遮掩base classes内的名称。我们可以通过
使用using声明式或转交函数（在devied class function中调用base版本进行实现）避免。

## 区分接口继承和实现继承
- 声明一个pure virtual函数的目的是为了让dericed classed只继承函数接口。
- 声明impure virtual函数的目的是为了让derived classes继承该函数的接口和缺省实现。
- 声明non-virtual函数的目的是为了令derived classes继承函数的接口以及一份强制性实现。


## 考虑virtual函数以外的其他选择
- 借由Non-Virtual Interface手法实现Template Method模式
- 借由Function Pointers实现Strategy模式
- 借由tr1::function完成Strategy模式
- 古典Strategy模式

## 绝不重新定义继承而来的non-virtual函数
　　重新定义继承而来的non-virtual破坏了继承的is-a关系，不变性凌驾特异性。

## 绝不重新定义继承而来的缺省参数值
　　缺省参数值是静态绑定，当你在调用一个定义于derived class内的virtual函数同时却使用base class为它所指定的
缺省参数值。

## 通过复合塑模出has-a或“根据某物实现出”　　
　　在应用域复合意味着has-a(有一个)，程序中的对象其实相当于你塑造的世界中的某些事物（人，汽车）。
在实现域复合意味着is-implemented-in-terms-of(根据某物实现出)，实现细节上的人工制品（缓冲去，互斥器，查找树）

## 明智而审慎地使用private继承
　　private继承意味is-implemented-in-terms-of, 他通常比复合级别低。private继承可以造成空白基类最优化EBO。
private继承不会讲一个derived class对象转换为一个base class对象。由private继承而来的所有成员会变成private属性。
private继承是一种实现技术，意味只有部分被继承，接口部分应略去。

## 明智而审慎地使用多重继承
　　多重继承比单一继承复杂，可能导致新的歧义性以及对virtual继承的需要。而virtual继承会增加大小，速度，初始化和赋值复杂度的成本。
多重继承适用于涉及“public 继承某个interface class”和”private"继承某个协助实现的class"的两相组合。

---
# 模板与泛型编程

## 了解隐式接口和编译期多态
　　classes和templates都支持接口和多态，对classes而言接口是显式的，以函数签名为中心，多态则是通过virtual函数发生于运行期。
对与templeate参数而言，接口是隐式的，通过template具现化和函数重载解析发生于编译期。

## 了解typename的双重意义
　　声明template参数时，前缀关键词class和typename可以互换。
请使用关键词typename标识嵌套从属类型名称,但不得在base class lists或member initialization list内以它作为base class修饰符。

##  学习处理模板化基类内的名称
　　可在derived class templates内通过this->指涉base class templates内的成员名称，或借由一个明白写出的base class资格修饰符完成。

## 将与参数无关的代码抽离templates
　　Templates生成多个classes和多个函数，所以任何template代码都不该与某个造成膨胀的template参数产生相依关系。
因非类型模板参数而造成的代码膨胀，往往可以消除，做法是以函数参数或class成员变量替换template参数。
因类型参数而造成的代码膨胀，往往可降低，做法是让带有完全相同二进制表述的具现类型共享实现码。

## 运用成员函数模板接受所有兼容类型
 　请使用member function templates生成可接受所有兼容类型的函数。
 如果你声明member templates用于"泛化copy构造"或“泛化assignment操作”，你还是需要声明正常的copy构造函数和copy assignment
 操作符。

## 需要类型转换时请为模板定义非成员函数
　　当我们编写一个class template， 而它所提供之“与此template相关的”函数支持“所有参数之隐式类型转换”时，请将那些函数定义为
“class template内部的friend函数”。

## 请使用traits classes表现类型信息
　　Traits classes使得“类型相关信息”在编译期可用，它们以templates和templeates特化完成实现。
整合重载技术后，traits classes有可能在编译期对类型执行if_else测试。

## 认识template元编程
　　模板元编程可将工作由运行期移往编译期，因而得以实现早期错误侦测和更高的执行效率。
TMP可被用来生成“基于政策选择组合”的客户定制代码，也可用来避免生成对某些特殊类型并不适合的代码。

---
# 定制new和delete

## 了解new-handler的行为
　　当operator new抛出异常以反映一个未获满足的内存需求之前，会先调用一个客户指定的错误处理函数，称为
new-handler.
一个良好的new-handler必须做到以下几点：
- 让更多内存可被使用
- 安装另一个new-handler
- 卸载new-handler
- 抛出bad_alloc
- 不返回
Nothrow new是一个颇为局限的工具，因为它只适用于内存分配，后继的构造函数调用还是可能抛出异常。

## 了解new和delete的合理替换时机
- 为了检测运用错误
- 为了收集动态分配内存之使用统计信息。
- 为了增加分配和归还的速度
- 为了降低缺省内存管理器带来的空间额外开销
- 为了补偿缺省分配器中的非最佳齐位
- 为了将相关对象成簇集中
- 为了获得非传统的行为

## 编写new和delete时需固守常规
　　operator new应该内含一个无穷循环，并在其中尝试分配内存，如果它无法满足内存需求，就该调用new-handler。它也
应该有能力处理0bytes申请，Class 专属版本则还应该处理“比正确大小更大的错误申请”。
　　operator delete应该在收到null指针时不做任何事。Class专属版本则还应该处理“比正确大小更大的错误申请”。

## 写了plac ement new也要写placement delete
　　如果operator new接受的参数除了一定会有的那个size_t之外还有其他，这个是一个placement new.

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
