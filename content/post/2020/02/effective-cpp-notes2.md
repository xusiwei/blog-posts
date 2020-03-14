---
title: "Effective C++（第三版）第二章笔记"
date: "2020-02-25T20:08:22+08:00"
draft: false
toc: true
tags: ["notes", "C++"]
categories: ["读书笔记"]
---

构造、析构、赋值运算符

## 5. 了解C++默默编写并调用哪些函数

一个空的类，如果你没有声明，编译器会为它声明：

1. 默认构造函数
2. 拷贝构造函数
3. 赋值运算符函数
4. 析构函数

例如，如下代码段中，你没有为Empty声明任何函数，但后面的拷贝构造、赋值代码都可以编译通过。

```c++
{
    class Empty {};
    Empty e1; // default constructor
    Empty e2(e1); // copy constructor
    e2 = 21;  // operator=
} // destructor
```

编译器生成的拷贝构造函数和默认赋值构造函数，只是单纯地将来源对象的各个数据成员拷贝到目标对象。
赋值运算符函数的这种默认版本，使得C++的类有了和C struct同样的语义。

**如果你为类定义了构造函数，编译器就不在为它生成默认构造函数**。

**当类或其基类含有无法赋值的数据成员时，赋值运算符函数不会自动生成**。例如，类中包含const类型或引用。

## 6. 若不想使用编译器自动生成的函数，就明确拒绝

C++98下的做法是：将其声明为private，并且没有实现。例如：

```c++
class House {
public:
    ...
private:
    ...
    House(const House&); // declarations only
    House& operator=(const House&);
};
```

这样可以阻止在类外拷贝House对象，但并不能阻止在类的其他成员函数中拷贝对象。
继而，引出，在基类中声明类似的拷贝构造函数和赋值运算符，就可以实现在子类的成员函数内也无法拷贝：

```c++
class Uncopyable {
protected: // allow construction
    Uncopyable() {} // and destruction of
    ~Uncopyable() {} // derived objects...

private:
    Uncopyable(const Uncopyable&); // ...but prevent copying
    Uncopyable& operator=(const Uncopyable&);
};
```

有了这样的`Uncopyable`之后，就可以通过继承实现子类的不可拷贝了。

```c++
class House: private Uncopyable { // class no longer
    ... // declares copy ctor or
};
```

你也可以使用boost提供的——noncopyable。

C++11下的做法则简单的多，直接使用 =delete; 修饰拷贝构造函数和赋值运算符函数即可。

## 7. 为多态基类声明virtual析构函数

这么做是为了使用父类指针操作子类对象，并且最终可能使用父类指针释放这个子类对象。例如有如下类型体系：

```c++
class TimeKeeper {
public:
    TimeKeeper();
    ~TimeKeeper();
    ...
};

class AtomicClock: public TimeKeeper { ... };
class WaterClock: public TimeKeeper { ... };
class WristWatch: public TimeKeeper { ... };
```

以及有如下使用的代码：

```c++
TimeKeeper *ptk = getTimeKeeper();  // get dynamically allocated object
    // from TimeKeeper hierarchy
... // use it
delete ptk; // release it to avoid resource leak
```

如果基类的析构函数没有声明为virtual，则其结果未定义。

> because C++ specifies that when a derived class object is deleted through a pointer to a base class with a non-virtual destructor, results are undefined.

通常，这种情况（父类析构不是virtual却用父类的指针释放子类对象），会导致实际调用的是父类的析构函数。

相反的是——**不被设计为基类或者说不被用于多态的类，就不用声明virtual析构函数**

PS: C++11的`final`关键字能够保护不被继承。

## 8. 别让异常逃离析构函数

若析构函数抛出异常，可能导致内存泄露或者其他的未定义行为。

处理手段：

1. 如果析构函数调用的某个函数可能会抛出异常，析构函数应该捕捉所有异常，吞下他们或者结束程序。
2. 如果客户需要对这个函数跑出的异常做出反应，那么应该提供一个普通函数（而非析构函数）。

## 9. 不在构造析构函数中调用virtual函数

Java/C#可以，C++不行。

因为基类的构造函数的执行早于派生类的构造函数，如果在基类中调用virtual成员函数下降到派生类中，
那么可能会访问未初始化的变量，所以C++不允许你这么做。

根本原因是：base class构造期间，对象的类型是base class而非derived class，包括运行时类型信息。
例如，在其中使用dynamic_cast或typeid，均会被视作base class。

同样的道理，基类的析构函数的执行晚于派生类的析构函数，如果在基类的析构函数中调用virtual成员函数下降到派生类，也可能会访问已经析构的数据成员。

## 10. 令`operator=`返回一个reference to `*this`

这是为了支持“链式赋值”：

```c++
int x, y, z;
x = y = z; // 链式赋值，等价于 x = (y = (z = 15));
```

## 11. 在`operator=`中处理自我赋值

```c++
class Widget { ... };
Widget w;
...
w = w; // assignment to self
```

对于资源管理类型，例如

```c++
class Bitmap { ... };
class Widget {
    ...
    private:
    Bitmap *pb; // ptr to a heap-allocated object
};

Widget&
Widget::operator=(const Widget& rhs) // unsafe impl. of operator=
{
    delete pb; // release current bitmap
    pb = new Bitmap(*rhs.pb); // start using a copy of rhs’s bitmap
    return *this; // see Item 10
}
```

如上类型在发生自我赋值时，`pb`和`rhs.pb`实际指向了同一个Bitmap对象，而`delete pb`会将其释放；
后面一行的解引用将会先发生释放后使用的问题（use-after-free）。

解决办法就是在函数的一开始检查：

```c++
Widget&
Widget::operator=(const Widget& rhs) // unsafe impl. of operator=
{
    if (&rhs == this) return *this;
    delete pb; // release current bitmap
    pb = new Bitmap(*rhs.pb); // start using a copy of rhs’s bitmap
    return *this; // see Item 10
}
```

## 12. 复制对象勿忘每一部分

当你编写一个copying函数：

1. 复制所有local成员变量
2. 调用base class对应的copying函数
