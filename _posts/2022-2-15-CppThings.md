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

## `move()` 函数

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

## `extern`, `static`声明

### extern
`extern`声明的变量，函数以及类型只能由外部（文件）引入，若未在其他文件声明（非`static`）则出现`Link Error`。

### static
**在`class`外部**表示只作用于文件内部，外部无权访问。(Only be visible to that `.cpp` file.)

**在`class`内部**，`static`成员独立于所有实例，为`class`中的唯一存在，需要以该类的全局变量来声明。
```c++
class A
{
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

class Entity
{
public:
    virtual std::string GetName()
    {
        return "Entity";
    }
    virtual std::string GetClassName() = 0; // pure virtual function, 子类继承时必须重载
};

class Player: public Entity
{
private:
    std::string myName;
public:
    Player(const std::string& name): myName(name) {}
    virtual std::string GetName() override
    {
        return myName;
    }
};

int main()
{
    Entity* e = new Entity(); // 编译出错, interface无法被实例化
    Player* p = new Player("Lyle"); // 编译出错, 继承时未重载pure virtual function
}
```

## Visibility (`private`, `public`, `protected`)

visibility的主要作用在于设定类的使用规则，避免带来复杂性。

`class`成员默认为`private`，`struct`成员默认为`public`。
`class`内部函数可以访问`private`或者`protected`成员，但子类只能访问父类的`protected`成员，无法访问其`private`成员。
```c++
#include <iostream>

class Entity
{
private:
    int x;
protected:
    void Print() {}
public:
    Entity()
    {
        x = 0; // 可访问
        Print(); // 可访问
    }
};

class Player: public Entity
{
public:
    Player()
    {
        x = 2; // 编译出错, 子类无法访问父类的private成员
        Print(); // 可访问
    }
};

int main()
{
    Entity* e;
    e.x = 2; // 编译出错, private成员无法从外部访问
    e.Print(); // 编译出错, protected成员同样无法从外部访问
}
```

## Arrays

```c++
#include <array>

class Entity
{
public:
    const int size = 5;
    int example[size]; // 编译出错, 必须是compile-time known constant

    static const int exampleSize = 5;
    int example[exampleSize]; // 正确

    std::array<int, exampleSize> arr;

    Entity()
    {
        for (int i = 0; i < arr.size(); ++i) // 区别于row array, 可调用size()方法
            arr[i] = 1;
    }
};
```

## String

```c++
#include <iostream>
#include <string>

int main()
{
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

int main()
{
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

class Entity
{
private:
    int m_X, m_Y;
    mutable int var;
public:
    int GetX() const // 此处const表示const方法
    {
        var = 2; // 被mutable标记的变量即使在const方法中依然可被改写
        return m_X; // 其他变量为只读
    }

    void SetX(int x)
    {
        m_X = x;
    }
};

void PrintEntity(const Entity& e)
{
    std::cout << e.GetX() << std::endl;
    // const实例只能调用const方法, 此处若GetX()为非const方法则编译出错
}

int main()
{
    Entity e;
    PrintEntity(e);
}
```

关于`mutable`，除上述使用情况以外，较常见的还有lambda函数传值时的使用：
```c++
int main()
{
    int x = 8;
    auto f = [=]() mutable
    { // []中为capture method. &(&x): by reference; =(x): by value
        x++; // 若传值方式为by reference, 则无需mutable
        std::cout << x << std::endl; // 9
    };
    f();
    // x: 8
}
```
此处`mutable`相当于
```c++
int main()
{
    int x = 8;
    auto f = [=]()
    {
        int y = x;
        y++;
        std::cout << y << std::endl;
    };
    f();
}
```

## 类构造函数初始化列表 Member Initializer Lists

```c++
class Example
{
private:
    int a;
    float b;
public:
    Example(): a(0), b(0.8) {}
};
```
此处`Example()`构造函数从结果上等价于
```c++
Example()
{
    a = 0;
    b = 0.8;
}
```
使用初始化列表的构造函数**显示的初始化类的成员**；反之则是**对类成员赋值**而没有进行显示的初始化。因此会有性能差异。

对非内置类型成员变量，为了避免两次构造，推荐使用类构造函数初始化列表。但有的时候必须用带有初始化列表的构造函数：

- **成员类型是没有默认构造函数的类**: 若没有提供显示初始化式，则编译器隐式使用成员类型的默认构造函数，若类没有默认构造函数，则编译器尝试使用默认构造函数将会失败。
- **`const`成员或`reference`类型的成员**: 因为`const`对象或`reference`类型只能初始化，不能对他们赋值。

初始化数据成员与对数据成员赋值的含义是什么？有什么区别？

首先把数据成员按类型分类并分情况说明:

- **内置数据类型，复合类型（指针，引用）**: 在成员初始化列表和构造函数体内进行，在性能和结果上都是一样的。
- **用户定义类型（类类型）**: 结果上相同，但是性能上存在很大的差别。因为类类型的数据成员对象在进入函数体前已经构造完成，也就是说在成员初始化列表处进行构造对象的工作，调用构造函数，在进入函数体之后，进行的是对已经构造好的类对象的赋值，又调用个拷贝赋值操作符才能完成（如果并未提供，则使用编译器提供的默认按成员赋值行为）。

对用户定义类型，例如
```c++
struct Node
{
    Node* next = nullptr;
    int val;
    Node() {}
    Node(int x): val(x) {}
};
```
能在类类型的数据成员对象进入函数体前就将其构造完成。
```c++
class Example
{
private:
    int a;
    Node node;
public:
    Example()
    {
        a = 0;
        node = Node(0); // this actually called constructor twice: 1. Node node = Node(); 2. node = Node(0);
    }
    Example(): a(0), node(Node(0)) /* or node(0) */ {} // faster
}
```
**注意:** 构造顺序应与声明顺序一致<!--（取决于编译器）-->。

## 实例化 Creating Instantiate Objects

区别于`Java`或`C#`（默认只能heap），`C++`的类在由stack和heap管理的内存上均可实例化。
在stack上构造实例性能上优于heap，在作用域结束时（scope外）相应内存会被自动释放。
```c++
#include <iostream>
#include <string>

using String = std::string;

class Entity
{
private:
    String myName;
public:
    Entity(): myName("Unknown") {}
    Entity(const String& name): myName(name) {}

    const String& GetName() const
    {
        return myName;
    }
};

int main()
{
    Entity entity1; // 使用默认构造函数在stack上构造
    Entity entity2("Lyle"); // 等价于 Entity entity = Entity("Lyle"); 在stack上构造

    Entity* e;
    {
        Entity entity3("Lyle");
        e = &entity3;
        std::cout << e->GetName() << std::endl; // "Lyle"
    }
    std::cout << e->GetName() << std::endl; // "" entity3被释放

    {
        Entity* entity3 = new Entity("Lyle"); // 使用`new` keyword为在heap上构造, 以指针形式返回
        e = entity3;
        std::cout << e->GetName() << std::endl; // "Lyle"
    }
    std::cout << e->GetName() << std::endl; // "Lyle"
    delete e; // 需要手动释放
}
```
必须在heap上构造的两种情况：
- 需要于**作用域外时**依然生效（need to control the life time）。
- **实例占用较大内存时**: stack的容量通常较小（1MB～2MB，取决于编译器和平台）。

### `new` Keyword

`new`是一个operator，在heap上分配内存，与`C`中的`malloc()`有相同行为，但是`new`调用了构造函数。

相对的，通过`new`构造的实例一般情况需要使用`delete`来释放，`delete`与`C`中的`free()`有相同行为，但是调用了`destructor`。
```c++
int main()
{
    int a = 2; // stack上申请连续的4byte
    int* b = new int[50]; // heap上申请连续的200byte; new[] 逐个调用constructor

    Entity* e = new Entity(); // new() 括号中可指定分配的地址
    Entity* e = (Entity*)malloc(sizeof(Entity)); // 与上面的唯一区别在于没有调用构造函数
    delete e; // 调用destructor
    free(e);

    delete[] b; // 释放时需要与new的方式对应, 逐个调用destructor
}
```

## 隐式转换 Implicit Conversion and `explicit` Keyword

编译器允许构造实例时通过构造函数进行**一次**隐式转换。
```c++
#include <iostream>
#include <string>

class Entity
{
private:
    std::string m_Name;
    int m_Age;
public:
    Entity(std::string& name): m_Name(name), m_Age(-1) {}

    explicit Entity(int age) // 此处`explicit` keyword表示通过该构造函数构造时必须以显式的方式
        : m_Name("Unknown"), m_Age(age) {}
};

void PrintEntity(const Entity& entity)
{
    // Do some printing
}

int main()
{
    Entity e = "Lyle"; // 发生隐式转换, 对应的显式构造为 e("Lyle")
    Entity e = 25; // 编译出错, 无法进行隐式转换

    PrintEntity(25); // 编译出错, 理由同上
    PrintEntity("Lyle"); // 编译出错, 字符串默认为`const char*`(char array)类型, 需要先进行一次隐式转换(cast to `std::string`), 再隐式转换成`Entity`类型, 而编译器只允许发生一次隐式转换
    PrintEntity(Entity("Lyle")); // or PrintEntity(std::string("Lyle")) 为正确构造方式
}
```

## Operator Overloading

`C++`允许类对操作符（如`+`，`-`，`*`，`/`，`==`等）进行重载。
```c++
#include <iostream>

struct Vector2
{
    float x, y;

    Vector2(float x, float y): x(x), y(y) {}

    Vector2 Add(const Vector2& other) const
    {
        return Vector2(x + other.x, y + other.y);
    }

    Vector2 Multiply(const Vector2& other) const
    {
        return Vector2(x * other.x, y * other.y);
    }

    Vector2 operator+(const Vector2& other) const // 对`+`重载
    {
        return Vector2(x + other.x, y + other.y); // (*this).Add(other)
    }

    Vector2 operator*(const Vector2& other) const // 对`*`重载
    {
        return Vector2(x * other.x, y * other.y); // (*this).Multiply(other)
    }

    bool operator==(const Vector2& other) const // 对逻辑运算重载
    {
        return x == other.x && y == other.y;
    }

    bool operator!=(const Vector2& other) const
    {
        return !(*this == other);
    }
};

std::ostream& operator<<(std::ostream& stream, const Vector2& vec) const // 对 stream(<<) 重载, 相当于通过对操作符重载实现`std::to_string()`
{
    stream << vec.x << ", " << vec.y;
    return stream;
}

int main()
{
    Vector2 position(3.0f, 3.0f);
    Vector2 speed(0.5f, 1.5f);
    Vector2 powerup(1.1f, 1.1f);

    Vector2 result = position.Add(speed.Multiply(powerup));
    Vector2 res = position + speed * powerup; // 与上面一致, 好处是增强可读性

    std::cout << res << std::endl;
    std::cout << speed == powerup << std::endl;
}
```

## 友元 友元类/友元函数

类的友元函数是定义在类外部，但有权访问类的所有`private`成员和`protected`成员。尽管友元函数的原型有在类的定义中出现过，但是友元函数并不是成员函数。

友元可以是一个函数，该函数被称为友元函数；友元也可以是一个类，该类被称为友元类，在这种情况下，整个类及其所有成员都是友元。

如果要声明函数为一个类的友元，需要在类定义中该函数原型前使用`friend` keyword；
如果要声明一个类的友元类，则需要在该类的定义中放置友元类的声明，如下所示：
```c++
#include <iostream>

class Box
{
private:
   double width;
public:
   double length;
   friend void printWidth(Box box); // 友元函数
   friend class BigBox; // 友元类
   void setWidth(double wid);
};

class BigBox
{
public :
    void Print(int width, Box &box)
    {
        // `BigBox`是`Box`的友元类, 它可以直接访问`Box`类的任何成员
        box.setWidth(width);
        std::cout << "Width of box : " << box.width << std::endl;
    }
};

// 成员函数定义
void Box::setWidth(double wid)
{
    width = wid;
}

// 注意: `printWidth()`不是任何类的成员函数
void printWidth(Box box)
{
    /* 因为`printWidth()`是`Box`的友元, 它可以直接访问该类的任何成员 */
    cout << "Width of box : " << box.width << endl;
}

int main()
{
    Box box;
    BigBox big;

    box.setWidth(10.0d);

    // 使用友元函数输出
    printWidth(box);

    // 使用友元类中的方法设置成员变量
    big.Print(20.0d, box);
}
```