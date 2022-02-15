---
layout:     post
title:      Things about C++
subtitle:   C++ 学习笔记
date:       2022-2-15
author:     Lyle
header-img: img/post-bg-eula.png
catalog: true
tags:
    - C++
    - Programing Language
---

## Pointer

```c++
int *arr[]; // arr是一个存指针的数组
int (*arr)[3]; // arr是一个指向[3](1个有3个int的数组)的指针
// declaration:
int (*arr)[3] = &(int []){1, 2, 3};
```

## 类构造函数初始化列表

```c++
class example {
private:
    int a;
    float b;
public:
    example(): a(0), b(0.8) {}
};
```
此处`example()`构造函数等价于
```c++
example() {
    a = 0;
    b = 0.8;
}
```
使用初始化列表的构造函数显示的初始化类的成员；反之则是对类成员赋值而没有进行显示的初始化。
对用户定义类型，例如
```c++
struct Node {
    Node* next = nullptr;
    int val;
    Node() {}
    Node(int x): val(x) {}
};
```
能在类类型的数据成员对象进入函数体前就将其构造完成。

## move() 函数

转移变量的值，与copy（赋值）不同。
```c++
#include <string>
#include <vector>

std::string foo = "foo-string";
std::string bar = "bar-string";
std::vector<std::string> vec;

vec.push_back(foo); // copy, foo仍然是"foo-string"
vec.push_back(move(bar)); // move, bar为空（未赋值状态）
```

## Interfaces In C++ (Pure Virtual Functions)

