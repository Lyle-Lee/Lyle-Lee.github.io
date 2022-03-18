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

`class`成员默认为`private`，`struct`成员默认为`public`。
`class`内部函数可以访问`private`或者`protected`成员，但子类只能访问父类的`protected`成员，无法访问其`private`成员。
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

## Arrays

```c++
#include <array>

class Entity {
public:
    const int size = 5;
    int example[size]; // 编译出错, 必须是compile-time known constant

    static const int exampleSize = 5;
    int example[exampleSize]; // 正确

    std::array<int, exampleSize> arr;

    Entity() {
        for (int i = 0; i < arr.size(); ++i) // 区别于row array, 可调用size()方法
            arr[i] = 1;
    }
};
```

## String

```c++
#include <iostream>
#include <string>

int main() {
    char* name = "Lyle"; // 默认为 const char* name = "Lyle";
    name[0] = 'l';
    std::cout << name << std::endl; // 执行出错(undefined behavior), 修改read-only memory

    std::string = "Lyle"; // 或者 char name[] = "Lyle";
    name[0] = 'l';
    std::cout << name << std::endl; // 正确
}
```

## Const

const variable
```c++
#include <iostream>

int main() {
    const int MAX_SIZE = 100;
    int* a = new int;
    const int* b = new int;
    int* const c = new int;

    *a = 2;
    a = (int*)&MAX_SIZE;
    *b = 2; // 编译出错, 内容为const, 无法改写
    b = (int*)&MAX_SIZE; // 正确
    *c = 2; // 正确
    c = (int*)&INT_SIZE; // 编译出错, 地址为const, 无法改写
}
```

const method
```c++
#include <iostream>

class Entity {
private:
    int m_X, m_Y;
    mutable int var;
public:
    int GetX() const { // 此处const表示const方法
        var = 2; // 被mutable标记的变量即使在const方法中依然可被改写
        return m_X; // 其他变量为只读
    }

    void SetX(int x) {
        m_X = x;
    }
};

void PrintEntity(const Entity& e) {
    std::cout << e.GetX() << std::endl;
    // const实例只能调用const方法, 此处若GetX()为非const方法则编译出错
}

int main() {
    Entity e;
    PrintEntity(e);
}
```

关于`mutable`，除上述使用情况以外，较常见的还有lambda函数传值时的使用：
```c++
int main() {
    int x = 8;
    auto f = [=]() mutable { // []中为capture method. &(&x): by reference; =(x): by value
        x++; // 若传值方式为by reference, 则无需mutable
        std::cout << x << std::endl; // 9
    };
    f();
    // x: 8
}
```
此处`mutable`相当于
```c++
int main() {
    int x = 8;
    auto f = [=]() {
        int y = x;
        y++;
        std::cout << y << std::endl;
    };
    f();
}
```

## 类构造函数初始化列表 Member Initializer Lists

```c++
class Example {
private:
    int a;
    float b;
public:
    Example(): a(0), b(0.8) {}
};
```
此处`Example()`构造函数从结果上等价于
```c++
Example() {
    a = 0;
    b = 0.8;
}
```
使用初始化列表的构造函数*显示的初始化类的成员*；反之则是*对类成员赋值*而没有进行显示的初始化。因此会有性能差异。

对非内置类型成员变量，为了避免两次构造，推荐使用类构造函数初始化列表。但有的时候必须用带有初始化列表的构造函数：

- *成员类型是没有默认构造函数的类*: 若没有提供显示初始化式，则编译器隐式使用成员类型的默认构造函数，若类没有默认构造函数，则编译器尝试使用默认构造函数将会失败。
- *`const`成员或`reference`类型的成员*: 因为`const`对象或`reference`类型只能初始化，不能对他们赋值。

初始化数据成员与对数据成员赋值的含义是什么？有什么区别？

首先把数据成员按类型分类并分情况说明:

- *内置数据类型，复合类型（指针，引用）*: 在成员初始化列表和构造函数体内进行，在性能和结果上都是一样的。
- *用户定义类型（类类型）*: 结果上相同，但是性能上存在很大的差别。因为类类型的数据成员对象在进入函数体前已经构造完成，也就是说在成员初始化列表处进行构造对象的工作，调用构造函数，在进入函数体之后，进行的是对已经构造好的类对象的赋值，又调用个拷贝赋值操作符才能完成（如果并未提供，则使用编译器提供的默认按成员赋值行为）。

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
```c++
class Example {
private:
    int a;
    Node node;
public:
    Example() {
        a = 0;
        node = Node(0); // this actually called constructor twice: 1. Node node = Node(); 2. node = Node(0);
    }
    Example(): a(0), node(Node(0)) /* or node(0) */ {} // faster
}
```
*注意:* 构造顺序应与声明顺序一致（取决于编译器）。