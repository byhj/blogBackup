title: Effective C++总结
date: 2015-12-17 21:11:28
tags: C++

---

# 让自己习惯C++

## 视C++为一个语言联邦
当今的C++是个多重范型编程语言(过程形式，面向对象形式，函数形式，泛型形式，元编程形式)，我们应该将C++视为由语言组成的联邦而非单一语言,C++主要分为以下四个次语言：

- C:                   C++部分继承了C的语法，这个部分没有模板，重载，继承，异常等语法。
- Object-Oriented C++: 包含classes(构造析构)， 封装，继承，多态。
- Template C++:        泛型编程的基础，带来了所谓的Template metaprogramming(TMP)编程。
- STL:                 STL是templater程序库，主要包括容器，迭代器，算法部分。

不同的次语言有着不同的规则，在编写程序时要按照你的情况进行策略的转变。
 <!--more-->

## 尽量以const, enum, inlne替代 #define


---

```
　static_cast<type>(expression)
```
