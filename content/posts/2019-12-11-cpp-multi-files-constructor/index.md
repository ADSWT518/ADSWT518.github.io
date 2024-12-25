---
title: 程序含有多个文件时构造函数的默认值问题
description: 初学C++时的一次debug记录
date: 2019-12-11
tags: [C++]
---

> 本文原载于 https://blog.csdn.net/qq_45377501/article/details/103499442 ，是大一初学C++时的一次debug记录。

如果想在定义对象时可以指定初值也可以不指定，可以在类中设计一个带默认值的构造函数。下面这个程序：

```cpp
//文件：main.cpp
#include "example.h"
#include <iostream>
using namespace std;

int main()
{
    hello A(2), B;
    A.print();
    B.print();
    return 0;
}

//文件：example.h
#ifndef EXAMPLE_H_INCLUDED
#define EXAMPLE_H_INCLUDED

class hello{
private:
    int b;
public:
    hello(int c);
    void print();
};
#endif // EXAMPLE_H_INCLUDED

//文件：example.cpp
#include "example.h"
#include <iostream>
using namespace std;
hello::hello(int c)
{
    b = c;
}

void hello::print()
{
    cout <<b;
}
```

在编译时会报错：

```bash
error: no matching function for call to 'hello::hello()'
```

是因为我们定义的构造函数必须要指定初值

而如果把类的执行文件改成这样：

```cpp
//文件：example.cpp
#include "example.h"
#include <iostream>
using namespace std;


hello::hello(int c=0)//加上了一个默认值
{
    b = c;
}

void hello::print()
{
    cout <<b;
}
```

却仍然出现了同样的错误：

```bash
error: no matching function for call to 'hello::hello()'
```

而如果不改变执行文件而改变头文件中构造函数的声明：

```cpp
//文件：example.h
#ifndef EXAMPLE_H_INCLUDED
#define EXAMPLE_H_INCLUDED

class hello{
private:
    int b;
public:
    hello(int c=0);//加上了一个默认值
    void print();
};
#endif // EXAMPLE_H_INCLUDED
```

则编译通过

而如果执行文件和头文件的构造函数都加上默认值：

```cpp
//文件：example.h
#ifndef EXAMPLE_H_INCLUDED
#define EXAMPLE_H_INCLUDED

class hello{
private:
    int b;
public:
    hello(int c=0);//加上了一个默认值
    void print();
};
#endif // EXAMPLE_H_INCLUDED

//文件：example.cpp
#include "example.h"
#include <iostream>
using namespace std;
hello::hello(int c=0)//加上了一个默认值
{
    b = c;
}

void hello::print()
{
    cout <<b;
}
```

则在编译时会报错：

```bash
error: default argument given for parameter 1 of 'hello::hello(int)' [-fpermissive]
```

这是因为函数参数的默认值不可以在声明和定义时都给出

当声明和定义在同一个文件中时，默认值可以在两个地方的任意一个地方给出；可是分开成两个文件时，就只能在声明时给出了。

再做尝试，将构造函数放入头文件中：

```cpp
//文件：example.h
#ifndef EXAMPLE_H_INCLUDED
#define EXAMPLE_H_INCLUDED

class hello{
private:
    int b;
public:
    hello(int c);
    void print();
};

hello::hello(int c=0)//定义处加上了一个默认值
{
    b = c;
}

void hello::print()
{
    cout <<b;
}
#endif // EXAMPLE_H_INCLUDED
```

编译通过

__思考：类的用户在使用类的时候主要是借助头文件来知道这个类是如何使用的，而不会去看执行文件。如果在执行文件中给出了参数的默认值，很有可能被用户所忽略。所以c++可能规定，必须在头文件中给出默认值，这是很有实用意义的。而如果写在一个文件中，用户也可以看到函数的定义，可能因此c++允许在定义时给出默认值。__
