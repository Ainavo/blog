---
title: C++ 中的模板
date: 2023-05-09 09:27:15
tags:
  - C++
---

在 C++ 中模板分为函数模板和类模板两种，函数模板用于生成函数，类模板用于生成类。

<!--more-->

# 函数模板

函数模板写法如下：

```
template <class 类型参数1, class 类型参数2, ...>
返回值类型 模板名（形参表）
{
    函数体
}
```

其中 `class` 的关键字可以用 `typename` 替换，例如：

```
template <typename 类型参数1, typename 类型参数2, ...>
```

example1:

```C++
template <typename T>
void Swap(T &x, T &y)
{
    T tmp = x;
    x = y;
    y = tmp;
}
```

其中，`T` 是类型参数，在编译时由模板对应的类型自动替换，当编译器编译到调用函数模板语句时， 会根据实参的类型判断如何替换模板中的类型参数。

```C++
#include <iostream>

template <typename T>
void Swap(T &x, T&y)
{
    T tmp = x;
    x = y;
    y = tmp;
}

int main()
{
    int x1 = 1;
    int y1 = 2;
    std::cout << Swap(x1, y1) << std::endl; //当输入参数为 int 时，自动调用 int 型函数，T--> int

    float x2 = 1.0;
    float y2 = 2.0;
    std::cout << Swap(x2, y2) << std::endl; //当输入参数为 float 时，自动调用 float 型函数 T--> float
}
```

编译器有模板自动生成函数的过程叫模板的实例化。有模板实例化的函数乘坐模板函数。在某些编译器中，只有模板被实例化时候才检查其语法的正确性，如果一个模板没有被调用，则不会报告模板中存在的语法错误。

# 类模板

类模板逻辑同类，使用同函数模板。
exp1：

```C++
    // member_function_templates1.cpp
    template<class T, int i>
    class MyStack
    {
        T*  pStack;
        T StackBuffer[i];
        static const int cItems = i * sizeof(T);
    public:
        MyStack( void );
        void push( const T item );
        T& pop( void );
    };

    template< class T, int i > MyStack< T, i >::MyStack( void )
    {
    };

    template< class T, int i > void MyStack< T, i >::push( const T item )
    {
    };

    template< class T, int i > T& MyStack< T, i >::pop( void )
    {
    };

    int main()
    {
    }
```

# 参考文献

[1] http://c.biancheng.net/view/315.html
[2] https://learn.microsoft.com/zh-cn/cpp/cpp/class-templates?view=msvc-170
