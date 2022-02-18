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

## extern, static声明

### extern
`extern`声明的变量，函数以及类型只能由外部（文件）引入，若未在其他文件声明（非`static`）则出现`Link Error`。

### static
**在`class`外部**表示只作用于文件内部，外部无权访问。(Only be visible to that `.cpp` file.)
**在`class`内部**，`static`成员独立于所有实例，为`class`中的唯一存在，需要以该类的全局变量来声明。
```c++
class A {
    static int x, y;
};

int A::x;
int A::y;
```
`static`方法同样作为该类所有实例的共通且唯一的方法而存在。
`static`方法无法访问非`static`成员（非`static`成员作为实例成员存在，对`static`方法来说相当于未定义）。

## Interfaces In C++ (Pure Virtual Functions)

包含pure virtual function的类只能被当作子类继承的模版，称为interface。
```c++
#include <iostream>
#include <string>

class Entity {
public:
    virtual std::string GetName() {
        return "Entity";
    }
    virtual std::string GetClassName() = 0; // pure virtual function, 子类继承时必须重载
};

class Player: public Entity {
private:
    std::string myName;
public:
    Player(const std::string& name): myName(name) {}
    virtual std::string GetName() override {
        return myName;
    }
};

int main() {
    Entity* e = new Entity(); // 编译出错, interface无法被实例化
    Player* p = new Player("Lyle"); // 编译出错, 继承时未重载pure virtual function
}
```

## Visibility (private, public, protected)

visibility的主要作用在于设定类的使用规则，避免带来复杂性。
`class`成员默认为`private`，`struct`成员默认为`public`。`class`内部函数可以访问`private`或者`protected`成员，但子类只能访问父类的`protected`成员，无法访问其`private`成员。
```c++
#include <iostream>

class Entity {
private:
    int x;
protected:
    void Print() {}
public:
    Entity() {
        x = 0; // 可访问
        Print(); // 可访问
    }
};

class Player: public Entity {
public:
    Player() {
        x = 2; // 编译出错, 子类无法访问父类的private成员
        Print(); // 可访问
    }
};

int main() {
    Entity* e;
    e.x = 2; // 编译出错, private成员无法从外部访问
    e.Print(); // 编译出错, protected成员同样无法从外部访问
}
```
