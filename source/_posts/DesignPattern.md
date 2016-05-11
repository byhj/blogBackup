title: Design Pattern
date: 2015-12-20 20:42:54
tags:
    - C++
    - Pattern

---

# Creational
创建型模式与对象的创建有关


Class
## Factory Method
定义一个用于创建对象的接口，让子类决定实例化哪一个类，使一个类的实例化推迟到其子类。
```
```

Object
## Abstract Factory
提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。
```
```
<!--more-->

## Builder
将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以产生不同的表示。
```
```

## Singleton
保证一个类仅有一个实例，并提供一个访问它的全局访问点。
```
```

## Prototype
用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象。
```
```

---

# Structural
处理类或对象的组合。

Class
## Adapter
```
```

Object
## Adapter
将一个类的接口转换成客户端希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。

```
class Target
{
public:
   virtual ~Target() = default;
   virtual std::string Operation() = 0;
};


class Adaptee
{
public:
   virtual  ~Adaptee() = default;
   std::string method()
   {
     return "Adapter";
   }
};

class Adapter: public Target, private Adaptee
{
public:
    string Opration() override
    {
      return method();
    }
};
```

## Bridge
将抽象部分与它的实现部分分离，使它们都可以独立地变化。
```
```

## Composite
将对象组合成树形结构以表示“部分-整体”的层次结构，使得用户对单个对象和组合对象的使用具有一致性。
```
```

## Decorator
动态地给一个对象添加一些额外的职责。
```
```

## Facade
为子系统中的一组接口提供一个一致的界面，定义了一个高层接口，使得子系统更加容易使用。
```
```

## Flyweight
运用共享技术有效地支持大量细粒度的对象。
```
```

## Proxy
为其他对象提供一种代理以控制对这个对象的访问。

```
```

---

# Behavioral
对类或对象怎么交互和怎样分配职责进行描述。

Class
## Template Method
　　定义一个操作中的算法的骨架，而将一些步骤延迟到子类中，使得子类可以不改变一个算法的结构即可
重定义该算法的某些特定步骤。

Object
## Chain Of Responsibility
　　使多个对象都有机会处理请求，从而避免请求的发送者和接受者之间的耦合关系，将这些对象连成一条链，
并沿着这条链传递该请求，直到有一个对象处理它为止。

## Command
　　将一个请求封装成一个对象，从而使你可用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可撤消的操作。


## Interpreter
　　给定一个句子，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子。

## Iterator
　　提供一种方法顺序访问一个聚合对象中的各个元素，而又不需暴露该对象的内部表示。

## Mediator
　　用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，
而且可以独立地改变它们之间的关系。

## Memento
在不破坏封装性的前提下，捕获一个对象的内部状态，并在该对象之外保存这个状态，这样以后就可将该对象恢复到原先保存的状态。

## Observer
定义对象间的一种一对多的依赖关系，当一个对象的状态发生变化时，所有依赖它的对象都能得到通知并被自动更新。

## State
允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。

## Strategy
定义一系列的算法，把它们一个个封装起来，并且使它们可相互替换，使得算法可独立于使用它的客户而变化。

## Visitor
表示一个作用于某对象结构中的各元素的操作，它使你可以在不改变各元素的类的前提下定义作用于这些元素的新操作。

---

[Source Code](https://github.com/byhj/DesignPattern.git)
