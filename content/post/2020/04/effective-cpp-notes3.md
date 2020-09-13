---
title: "Effective C++（第三版）第三章笔记"
date: "2020-04-19T21:28:53+08:00"
draft: false
toc: true
tags: []
categories: []
---

资源管理

## 13. 以对象管理资源

* RAII（Resource Acquisition Is Initialization），通俗的说就是用类的构造函数和和析构函数管理资源。
* C++98 有 auto_ptr （C++17中被废弃，不再能够使用）
* C++11 有 unique_ptr （用于取代auto_ptr），shared_ptr 、weak_ptr （用于避免 shared_ptr 的循环引用）

## 14. 资源管理类中小心copying行为

表示不同资源的RAII类，对于拷贝行为的需求也不同：

* 禁止复制。很多实际对象“被复制”是不合理的，因此需要禁止复制。C++98中将对应函数声明为`private`实现，C++11中有`=delete`修饰。
* 引用计数。直到最后一个使用者停止使用，才释放相应资源。`std::shared_ptr`即带有引用计数的智能指针。
* 复制资源。例如字符串对象的复制，可能希望内容也被复制一份用于后续的修改，即所谓的深拷贝（deep-copying）。在拷贝构造、拷贝赋值函数中实现底层资源的拷贝。
* 转移所有权。C++98中auto_ptr的复制动作是这一语义。C++11引入了移动构造函数（move constructor）和移动赋值运算符函数（move operator assignment）之后，这才是已于区分和实现的。

## 15. 在资源管理类中提供对原始资源的访问

* 标准库中的智能指针都提供了 `get()` 成员函数，用于取出原始指针。
* 标准库中的`std::unique_ptr`更进一步提供了`release()`成员函数，取出原始指针的同时还会交出控制权。

## 16. 成对使用new和delete时要采取相同的形式

目的——**为了防止内存泄露**

* `new`和`delete`对应，如`p = new T;`则要`delete p;`
* `new[]`和`delete[]`对应，如`p = new T[N];`则要`delete [] p;`

## 17. 以独立语句将new创建的对象放入智能指针

这里是说，如果使用智能指针管理new创建对象，就要把new表达式和智能指针创建放在一条语句中；

例如有如下测试程序：

```cpp
#include <stdio.h>
#include <iostream>
#include <memory>
#include <string>
#include <stdexcept>

struct Foo {
    Foo(int n)
        : data(n)
    {
        printf("Foo_%p {data: %d} born!\n", this, data);
    }

    ~Foo()  { printf("Foo_%p {data: %d} die!\n", this, data); }

    int data;
};

int GetPriority(int n)
{
    static const int priorities[]{20, 21, 22};
    if (n < 0) {
        throw std::invalid_argument("n invalid: " + std::to_string(n));
    } else if (n > sizeof(priorities)/sizeof(priorities[0])) {
        throw std::out_of_range("n out of range: " + std::to_string(n));
    }
    return priorities[n];
}

void Process(const std::shared_ptr<Foo>& foo, int priority)
{
}

int main()
{
    std::shared_ptr<Foo> p0{new Foo(0)};

    try {
        std::shared_ptr<Foo> p1{new Foo(-1)};
        Process(p1, GetPriority(p1->data));
    } catch (std::exception& e) {
        std::cout << "exception raised:" << e.what() << std::endl;
    }

    try {
        Foo* p2 = new Foo(-2);
        Process(std::shared_ptr<Foo>(new Foo(-2)), GetPriority(p2->data));
    } catch (std::exception& e) {
        std::cout << "exception raised:" << e.what() << std::endl;
    }
    return 0;
}
```

这个程序的输出是：

```txt
Foo_0x56004c1f6e70 {data: 0} born!
Foo_0x56004c1f72c0 {data: -1} born!
Foo_0x56004c1f72c0 {data: -1} die!
exception raised:n invalid: -1
exception raised:n invalid: -2
Foo_0x56004c1f6e70 {data: 0} die!
```
