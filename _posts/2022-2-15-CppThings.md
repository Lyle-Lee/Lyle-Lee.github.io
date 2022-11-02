---
layout:     post
title:      Things about C++
subtitle:   C++ 学习笔记
date:       2022-2-15
author:     Lyle
header-img: img/post-bg-lantern.JPG
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

## `extern`, `static`声明

### extern

`extern`声明的变量，函数以及类型只能由外部（文件）引入，若未在其他文件声明（非`static`）则出现`Link Error`。

### static

静态变量存放与内存分区的**全局/静态区**，在编译阶段会被初始化（若没有显式初始化则会被自动进行，如`int`型变量自动初始为`0`）。

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
`static`方法没有`this`指针，无法访问非`static`成员（非`static`成员作为实例成员存在，对`static`方法来说相当于未定义）。

## Virtual Functions

虚函数引入动态绑定，通过虚函数表（虚表，vtable，位于内存分区的**常量区**）来实现编译。虚表包含对基类中所有虚函数的映射（保存虚函数地址），对于有虚函数的类，编译器在编译阶段会自动生成属于该类的虚表。

```c++
#include <iotream>
#include <string>

class Entity
{
public:
    virtual std::string GetName() { return "Entity"; }
};

class Player: public Entity
{
private:
    std::string m_Name;
public:
    Player(const std::string& name): m_Name(name) {}
    std::string GetName() override { return m_Name; }
};

void PrintName(Entity* entity)
{
    std::cout << entity->GetName() << std::endl;
}

int main()
{
    Entity* e = new Entity();
    Player* p = new Player("Lyle");
    PrintName(e); // 打印"Entity"
    PrintName(p); // Player is an Entity, 打印"Lyle"
}
```

在含有虚函数的类对象被实例化时，对象地址的前4个字节存储指向虚表的指针vptr。
调用关系：`this`->vptr->vtable->virtual function。

### 多态

虚函数的引入实现了多态，其过程如下：

- 编译器在发现基类中有虚函数时，会自动为每个含有虚函数的类生成一份虚表，该表是一个一维数组，虚表里保存了虚函数的入口地址。
- 编译器会在每个对象的前4个字节中保存一个虚表指针，即vptr，指向对象所属类的虚表。在构造时，根据对象的类型去初始化虚指针vptr，从而让vptr指向正确的虚表，从而在调用虚函数时，能找到正确的函数。
- 在子类对象被实例化时，程序运行会自动调用构造函数，在构造函数中创建虚表并对虚表初始化。在构造子类对象时，会先调用父类的构造函数，此时，编译器只“看到了”父类，并将其作为父类对象初始化虚表指针，令它指向父类的虚表；当调用子类的构造函数时，再作为子类对象初始化虚表指针，令它指向子类的虚表。
- 当子类没有重写基类的虚函数时，其虚表指针指向的是基类的虚表；反之，其虚表指针指向的是自身的虚表；当子类中有自己的虚函数时，在自己的虚表中将此虚函数地址添加在后面。

## Interfaces In `C++` (Pure Virtual Functions)

纯虚函数只有定义没有实现，需要子类具体实现（需要实例化的类在继承时必须重写纯虚函数，对其进行实现）。
包含纯虚函数的类只能被当作接口，称为**interface/抽象类**。

```c++
#include <iostream>
#include <string>

class Printable
{
public:
    virtual std::string GetClassName() = 0; // pure virtual function, 子类继承时必须重写
};

class Entity: public Printable
{
public:
    virtual std::string GetName() { return "Entity"; }
    std::string GetClassName() override { return "Entity"; } // 必须重写，否则该类型无法实例化
};

class Player: public Entity
{
private:
    std::string m_Name;
public:
    Player(const std::string& name): m_Name(name) {}
    std::string GetName() override { return m_Name; }
    std::string GetClassName() override { return "Player"; } // 此处为非必须，因为父类`Entity`已重写纯虚函数
};

void PrintClassName(Printable* obj) // 作为接口，保证该基于该类型的子类实例有`GetClassName`这一特性
{
    std::cout << obj->GetClassName() << std::endl;
}

int main()
{
    Printable* pvf = new Printable(); // 编译出错, interface无法被实例化
    Entity* e = new Entity();
    Player* p = new Player("Lyle");
    PrintClassName(e); // "Entity"
    PrintClassName(p); // "Player"
}
```

**作用：** 如上面的`PrintClassName()`函数所示，通过定义`Printable`抽象类作为接口，保证了相应的子类有对其纯虚函数的重写，从而保证相应方法能够被调用，无视实际`class`类型。

对于析构函数为纯虚析构函数的抽象类，其每一个派生类析构函数会被编译器加以扩张，以静态调用的方式调用其每一个虚基类以及上一层基类的析构函数。因此强制了每一次继承都必须定义析构函数。

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
    int example[size]; // 编译出错, stack上分配内存必须是compile-time-known constant

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
    char* name = "Lyle"; // 默认为 const char* name = "Lyle";
    name[0] = 'l';
    std::cout << name << std::endl; // 执行出错(undefined behavior), 修改read-only memory

    std::string = "Lyle"; // 或者 char name[] = "Lyle";
    name[0] = 'l';
    std::cout << name << std::endl; // 正确
}
```

**注意：**`std::string`默认构造时为heap allocation。

## Const

`const` variable

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
    *b = 2; // 编译出错, 内容为const, 无法改写
    b = (int*)&MAX_SIZE; // 正确
    *c = 2; // 正确
    c = (int*)&INT_SIZE; // 编译出错, 地址为const, 无法改写
}
```

`const` method

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
        var = 2; // 被mutable标记的变量即使在const方法中依然可被改写
        return m_X; // 其他变量为只读
    }

    void SetX(int x)
    {
        m_X = x;
    }
};

void PrintEntity(const Entity& e)
{
    std::cout << e.GetX() << std::endl;
    // const实例只能调用const方法, 此处若GetX()为非const方法则编译出错
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

### `constexpr` vs. `const`

`constexpr`表示*constant expression*，基本概念与`const`一致，但`constexpr`可以用来声明构造函数，并在可能的情况下，要求编译器在编译阶段进行初始化。

虽然在运行前初始化有助于提升性能，但`constexpr`有很多限制，比如不能用来修饰虚函数（编译阶段无法决定），有虚基类的子类的构造函数不能为`constexpr`等。

```c++
constexpr float x = 42.0;
constexpr float y{108};
constexpr float z = exp(5, 3);
constexpr int i; // Error! Not initialized
int j = 0;
constexpr int k = j + 1; //Error! j not a constant expression
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
    std::cout << "Width of box : " << box.width << std::endl;
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

## `this`指针

`this`指针是所有成员函数的隐含参数，每一个对象都能通过`this`指针来访问自己的地址。因此，在成员函数内部，它可以用来指向调用对象。

友元函数没有`this`指针，因为友元不是类的成员。
```c++
#include <iostream>

void PrintEntity(Entity* e);

class Entity
{
public:
    int x, y;

    Entity(int x, int y)
    {
        // 非`const`成员函数中, `this`被定义为`Entity* const this`
        this->x = x;
        this->y = y;

        PrintEntity(this); // 在成员函数内部调用外部函数时, `this`可作为代表自身的入参
    }

    int GetX() const
    {
        // 在`const`成员函数中, `this`被定义为`const Entity* const this`
        return this->x;
    }
};

void PrintEntity(Entity* e)
{
    std::cout << e->y << std::endl;
}
```

## Smart Pointers

### Unique Pointer `std::unique_ptr`

本质是在stack上分配的指针，在scope结束时会自动调用对象的destructor。因此使用`unique_ptr`构造实例时，在使用`new`在heap上构造的同时无需调用`delete`来手动释放内存。

同样也是这个原因，为了防止memory leak，`unique_ptr`无法被拷贝。

```c++
#include <iostream>
#include <string>
#include <memory>

class Entity
{
public:
    Entity()
    {
        std::cout << "Created Entity!" << std::endl;
    }

    ~Entity()
    {
        std::cout << "Destroyed Entity!" << std::endl;
    }

    void Print() {}
};

class ScopedPtr() // `unique_ptr`的行为与此类相似
{
private:
    Entity* m_Ptr;
public:
    ScopedPtr(Entity* ptr): m_Ptr(ptr) {}

    ~ScopedPtr()
    {
        delete m_Ptr;
    }
}

int main()
{
    {
        ScopedPtr e = new Entity(); // 隐式转换
    } // scope结束后`e`被自动释放

    {
        std::unique_ptr<Entity> e(new Entity()); // `unique_ptr`不允许隐式转换, 其拷贝构造函数与赋值操作符均被delete
        e->Print(); // 可以像正常的`Entity`类指针一样调用其成员函数

        std::unique_ptr<Entity> entity = std::make_unique<Entity>(); // 更安全的使用方法, exception safety (C++14以后)
    } // scope结束后`e`, `entity`都被自动释放

    std::cin.get();
}
```

### Shared Pointer `std::shared_ptr` 与 Weak Pointer `std::weak_ptr`

shared pointer通过reference counting实现，构造时会额外分配一块内存给control block用于保存reference counting, 可以被赋值or拷贝。

而shared pointer赋值or拷贝给weak pointer时不会增加reference counting。

```c++
int main()
{
    std::weak_ptr<Entity> e2;
    {
        std::shared_ptr<Entity> e1;
        {
            std::shared_ptr<Entity> e(new Entity()); // 本质为两次构造, 有性能损耗
            std::shared_ptr<Entity> sharedEntity = std::make_shared<Entity>(); // 推荐使用, 考虑exception safety的同时一次性分配内存

            e1 = sharedEntity; // 可被拷贝, reference counting = 2
            e2 = sharedEntity; // 不计数, reference counting仍然为2
        } // `sharedEntity`的scope结束, reference counting = 1
    } // reference counting = 0, `Entity`对象被自动释放, 同时weak_ptr `e2`失效
}
```

### 这些智能指针分别是如何实现的

- `scoped_ptr`私有化了拷贝构造函数和赋值操作符，资源的所有权无法进行转移，也无法在容器中使用，这种方式杜绝了浅拷贝的发生。
- `unique_ptr`删除了拷贝构造函数和赋值操作符，因此不支持普通的拷贝或赋值操作。但引入了移动构造函数和移动操作符。所以它们保证了有唯一的智能指针持有此资源。`unique_ptr`还提供了`reset()`重置资源，`swap()`交换资源等函数，也经常会使用到。
- `shared_ptr`称为强智能指针，它的资源引用计数器在内存的heap上（这保证了，每个智能指针的引用计数变量会动态的变化）。通常用于管理对象的生命周期。只要有一个指向对象的`shared_ptr`存在，该对象就不会被析构。
- `weak_ptr`被称为弱智能指针，其对资源的引用不会引起资源的引用计数的变化，通常作为观察者，用于判断资源是否存在，并根据不同情况做出相应的操作。比如使用`weak_ptr`对资源进行弱引用，当调用`weak_ptr`的`lock()`方法时，若返回`nullptr`，则说明资源已经不存在，放弃对资源继续操作。否则，将返回一个`shared_ptr`对象，可以继续操作资源。另外，一旦最后一个指向对象的`shared_ptr`被销毁（引用计数器归0），对象就会被释放。即使有`weak_ptr`指向对象，对象也还是会被释放。当需要多个智能指针指向同一个资源时，使用带引用计数的`shared_ptr`。每增加一个智能指针指向同一资源，资源引用计数加1，反之减1。当引用计数为0时，由最后一个指向资源的智能指针将资源进行释放。

### 如何避免循环引用

```c++
#include <iostream>
#include <memory>

using namespace std;

class B; // 前置声明类B

class A
{
public:
    A() { cout << "A()" << endl; }
    ~A() { cout << "~A()" << endl; }
    shared_ptr<B> _ptrb; // 指向B对象的智能指针
};

class B
{
public:
    B() { cout << "B()" << endl; }
    ~B() { cout << "~B()" << endl; }
    shared_ptr<A> _ptra; // 指向A对象的智能指针
};

int main()
{
    shared_ptr<A> ptra(new A()); // ptra指向A对象，A的引用计数为1
    shared_ptr<B> ptrb(new B()); // ptrb指向B对象，B的引用计数为1
    ptra->_ptrb = ptrb;          // A对象的成员变量_ptrb也指向B对象，B的引用计数为2
    ptrb->_ptra = ptra;          // B对象的成员变量_ptra也指向A对象，A的引用计数为2

    cout << ptra.use_count() << endl; // 打印A的引用计数结果:2
    cout << ptrb.use_count() << endl; // 打印B的引用计数结果:2

    /*
	出main函数作用域，ptra和ptrb两个局部对象析构，分别给A对象和
	B对象的引用计数从2减到1，达不到释放A和B的条件（释放的条件是
	A和B的引用计数为0），因此造成两个new出来的A和B对象无法释放，
	导致内存泄露，这个问题就是“强智能指针的交叉引用(循环引用)问题”
	*/
    return 0;
}
```

**解决办法:** 这也是强弱智能指针的一个重要应用规则：定义对象时，用强智能指针`shared_ptr`，在其它地方引用对象时，使用弱智能指针`weak_ptr`。

```c++
#include <iostream>
#include <memory>

using namespace std;

class B; // 前置声明类 B

class A
{
public:
    A() { cout << "A()" << endl; }
    ~A() { cout << "~A()" << endl; }
    weak_ptr<B> _ptrb; // 指向 B 对象的弱智能指针。引用对象时，用弱智能指针
};

class B
{
public:
    B() { cout << "B()" << endl; }
    ~B() { cout << "~B()" << endl; }
    weak_ptr<A> _ptra; // 指向 A 对象的弱智能指针。引用对象时，用弱智能指针
};

int main()
{
    // 定义对象时，用强智能指针
    shared_ptr<A> ptra(new A()); // ptra 指向 A 对象，A 的引用计数为 1
    shared_ptr<B> ptrb(new B()); // ptrb 指向B 对象，B 的引用计数为 1
    // A 对象的成员变量 ptrb 也指向 B 对象，B 的引用计数为 1，因为是弱智能指针，引用计数没有改变
    ptra->_ptrb = ptrb;
    // B 对象的成员变量 ptra 也指向 A 对象，A 的引用计数为 1，因为是弱智能指针，引用计数没有改变
    ptrb->_ptra = ptra;

    cout << ptra.use_count() << endl; // 打印结果: 1
    cout << ptrb.use_count() << endl; // 打印结果: 1

    /*
	出 main 函数作用域，ptra 和 ptrb 两个局部对象析构，分别给 A 对象和
	B 对象的引用计数从 1 减到 0，达到释放 A 和 B 的条件，因此 new 出来的 A 和 B 对象
	被析构掉，解决了“强智能指针的交叉引用(循环引用)问题”
	*/
    return 0;
}
```

## 深/浅拷贝 拷贝构造函数

当类成员有指针时，默认的拷贝构造函数只能实现浅拷贝，只拷贝相应的指针并使其指向相同的一块内存。
这种情况下，两个对象被析构时会调用两次析构函数，而作为成员变量的指针指向的内存也会被释放两次，释放不属于自己的内存导致程序崩溃。
以一个自定义实现的`String`类为例：

```c++
#include <iostream>

class String
{
private:
    char* m_Buffer;
    unsigned int m_Size;
public:
    String(const char* string)
    {
        m_Size = strlen(string);
        m_Buffer = new char[m_Size + 1];
        memcpy(m_Buffer, string, m_Size);
        m_Buffer[m_Size] = 0; // end of a string ('\0')
    }

    String(const String& other) // 拷贝构造函数, 默认只拷贝自己的成员变量
        : m_Size(other.m_Size)
    {
        // memcpy(this, &other, sizeof(String)); // 默认情况, 相当于`m_Buffer(other.m_Buffer), m_Size(other.m_Size)`, 需要手动实现深拷贝
        m_Buffer = new char[m_Size + 1];
        memcpy(m_Buffer, other.m_Buffer, m_Size + 1); // 包含终止符
    }

    ~String()
    {
        delete[] m_Buffer; // 用new在heap上分配内存需要手动对其释放
    }

    char& operator[](unsigned int index)
    {
        return m_Buffer[index];
    }

    friend std::ostream& operator<<(std::ostream& stream, const String& string);
};

std::ostream& operator<<(std::ostream& stream, const String& string)
{
    stream << string.m_Buffer;
    return stream;
}

int main()
{
    String str1 = "Lyle";
    String str2 = str1;

    str2[0] = 'l';

    std::cout << str1 << std::endl; // "Lyle"
    std::cout << str2 << std::endl; // "lyle", `str2`与`str1`有了互相独立的内存, 在作用域结束后两者的析构都能够正常进行

    std::cin.get();
}
```

## Template

函数和类中都可引入模板，函数模板的实例化是由编译程序在处理函数调用时自动完成的，而类模板的实例化必须由程序员在程序中显式地指定。
两者都是在编译时生成额外的代码（编译前可视为不存在，取决于编译器）。

```c++
#include <iostream>
#include <string>

template<typename T>
void Print(T value)
{
    std::cout << value << std::endl;
}

template<typename T, int N>
class Array
{
private:
    T m_Array[N]; // `N`可作为compile-time-known类型在stack上分配内存
public:
    int GetSize() const { return N; }
};

int main()
{
    Print(5); // 函数模板可隐式调用
    Print<std::string>("Lyle"); // 显式

    Array<int, 5> arr1; // 必须显式
    Array<std::string, 5> arr2;
}
```

函数模板允许隐式调用和显式调用而类模板只能显式调用（在使用时类模板必须在`<>`中给出实际内容，而函数模板不必）。

## 函数指针

函数指针可以使程序访问CPU指令的地址（内存分区中的**代码区**），命名方式为`return_type(*name)(parameter_type)`。

```c++
#include <iostream>

void HelloWorld()
{
    std::cout << "HelloWorld" << std::endl;
}

int main()
{
    auto function = HelloWorld; // 本质为`&HelloWorld`(发生了隐式转换), 此时将函数地址(二进制CPU指令的地址)通过指针赋值给了`function`

    void(*function)() = HelloWorld; // 常规命名方式

    typedef void(*HelloWorldFunc)(); // 可以通过这种方式使命名方式与其他类型相同
    HelloWorldFunc function = HelloWorld;

    function(); // 可直接调用
}
```

**函数指针的作用:** 将函数作为其他函数（比如API中的函数）的入参，Lambda函数的应用场景。

```c++
#include <iostream>
#include <vector>

void PrintValue(int value)
{
    std::cout << "Value: " << value << std::endl;
}

void ForEach(const std::vector<int>& values, void(*func)(int))
{
    for (int value : values)
        func(value);
}

int main()
{
    std::vector<int> vec = { 1, 2, 3, 4, 5 };
    ForEach(vec, PrintValue); // 相当于告诉一个函数我想做什么事情

    ForEach(vec, [](int value) { std::cout << "Value: " << value << std::endl; }); // 应用Lambda函数
}
```

## Moving Semantics 移动赋值操作 移动构造函数

构造对象时产生的临时变量默认通过拷贝的方式传递给成员变量，此时调用拷贝构造函数会产生额外的性能开销，通过定义移动构造函数及引动赋值操作符能够避免不必要的拷贝。

```c++
#include <iostream>

class String
{
private:
    char* m_Buffer;
    uint32_t m_Size;
public:
    String() = default;
    String(const char* string)
    {
        std::cout << "Created" << std::endl;
        m_Size = strlen(string);
        m_Buffer = new char[m_Size + 1];
        memcpy(m_Buffer, string, m_Size);
        m_Buffer[m_Size] = 0; // end of a string ('\0')
    }

    String(const String& other) // 拷贝构造函数
        : m_Size(other.m_Size)
    {
        std::cout << "Copied" << std::endl;
        m_Buffer = new char[m_Size + 1];
        memcpy(m_Buffer, other.m_Buffer, m_Size + 1); // 包含终止符
    }

    String(String&& other) noexcept // 移动构造函数
    {
        std::cout << "Moved" << std::endl;
        m_Size = other.m_Size;
        m_Buffer = other.m_Buffer;

        other.m_Size = 0;
        other.m_Buffer = nullptr; // 把`other`变成虚空对象, 通过这种方式使临时对象被析构时不会把原先分配的内存释放, 从而使`other.m_Buffer`管理的资源由`this->m_Buffer`接管
    }

    ~String()
    {
        std::cout << "Destroyed" << std::endl;
        delete[] m_Buffer; // 用new在heap上分配内存需要手动对其释放
    }

    char& operator[](unsigned int index)
    {
        return m_Buffer[index];
    }

    friend std::ostream& operator<<(std::ostream& stream, const String& string);
};

std::ostream& operator<<(std::ostream& stream, const String& string)
{
    stream << string.m_Buffer;
    return stream;
}

class Entity
{
private:
    String m_Name;
public:
    Entity(const String& name): m_Name(name) {}
    Entity(String&& name): m_Name((String&&)name) {} // 通过右值引用的方式构造, 需要显式的cast才能正确调用成员变量的移动构造函数

    void PrintName()
    {
        std::cout << m_Name << std::endl;
    }
};

int main()
{
    // 定义移动构造函数前:
    Entity e1 = Entity(String("Lyle")); // 默认构造方式为先构造一个临时的`String`类型, 然后`Entity`类型的构造函数将其拷贝给成员变量`m_Name`
    // "Created" "Copied" "Destroyed" // 临时对象先被析构
    e1.PrintName(); // "Lyle"

    // 定义移动构造函数后:
    Entity e2 = Entity(String("Lyle")); // 避免了调用拷贝构造函数, 从而通过减少一次内存分配来提升性能
    // "Created" "Moved" "Destroyed"
    e2.PrintName(); // "Lyle"
}
```

### `std::move()` 函数

转移变量的值，移动语义。通过将对象cast成临时变量（右值引用），使得相应的类对象通过移动构造函数被实例化。

```c++
#include <string>
#include <vector>

std::string foo = "foo-string";
std::string bar = "bar-string";
std::vector<std::string> vec;

vec.push_back(foo); // copy, foo仍然是"foo-string"
vec.push_back(std::move(bar)); // move, bar为空（未赋值状态）
```

## Type Puning

利用`C`系语言的类型系统，我们可以强制编译器以不同方式去理解内存中的同一块区域，这种强制转换的方式不改变对象实际的二进制表示，称为**Type Puning**。

```c++
#include <iostream>

int main()
{
    int a = 50;
    double d = a;
    std::cout << d << std::endl; // 50.0, 此处为一般的类型转换, 编译器自动改变了对应二进制的表示, 使最终的值不变

    double d_pun = *(double*)&a;
    std::cout << d_pun << std::endl; // -9.25596e+61, 值改变但二进制表示不变

    return 0;
}
```

**注意：**type puning对象为引用的情况下存在风险，尤其当尝试解释所需要的内存不属于原对象时。例如：

```c++
double& d_pun = *(double*)&a;
d_pun = 0.0; // 尝试写入8 Byte而原对象仅持有4 Byte, run time error
```

**作用：**在合法的情况下将结构体等其他类型（内存分布与数组一致）以数组的方式读取。在想要创建一个包含结构体所有数据的数组时，type puning可以直接将原对象作为数组而无需额外时空开销（整个过程仅生成一个指针）。

```c++
#include <iostream>

struct Entity
{
    int x, y;

    int* GetPosition()
    {
        return &x;
    }
};

int main()
{
    Entity e = {5, 8};
    int* pos = (int*)&e;

    std::cout << pos[0] << ", " << pos[1] << std::endl; // 5, 8

    // more intuitively,
    pos = e.GetPosition();
    pos[1] = 10;

    return 0;
}
```

### `union`

当对象允许以不同方式被读取（以不同方式理解同一段内存）时，`union`可用于定义其允许的类型理解方式。允许匿名定义，但匿名无法声明内部函数。

```c++
#include <iostream>

struct Union
{
    union
    {
        float a;
        int b;
        // a和b占用同一段内存
    };
};

int main()
{
    Union u;
    u.a = 2.0f;

    std::cout << u.a << ", " << u.b << std::endl; // 2, 1073741824

    return 0;
}
```

**作用：**当我们需要以某种结构体或者类的方式去读取一个其他不同的结构体或者类时，通过定义`union`可在不创建类对象的情况下完成读取。

```c++
#include <iostream>

struct Vec2
{
    float x, y;
};

struct Vec4
{
    // float x, y, z, w; // normal case
    union
    {
        struct
        {
            float x, y, z, w;
        };
        struct
        {
            Vec2 a, b;
        };
    };
};

void PrintVec2(const Vec2& vec)
{
    std::cout << vec.x << ", " << vec.y << std::endl;
}

int main()
{
    Vec4 vec = {1.0f, 2.0f, 3.0f, 4.0f};
    PrintVec2(vec.a); // 以vec2形式读取

    vec.w = 5.0f; // 以vec4形式读取
    PrintVec2(vec.b); // 3, 5

    return 0;
}
```

## `C++` Style Type Casting

所做的操作与`C` style cast相同，除了合法性检查（complie / run time checking）以外无其他额外的操作。
虽有略微性能开销，但相较于`C` style cast以及type puning更安全。

### `static_cast`与`dynamic_cast`

`static_cast`在非法情况下无法通过编译（compile time checking），而`dynamic_cast`应用RTTI（Run Time Type Info），在非法情况下返回一个`nullptr`。

```c++
#include <iostream>

class Base {};

class Derived : private Base { // Inherited private/protected not public
};

int main()
{
	Derived d;
	Base* b = (Base*)(&d); // allowed
	Base* b1 = static_cast<Base*>(&d); // compile error: 'Base' is an inaccessible base of 'Derived'

	return 0;
}
```

`dynamic_cast`以多态为前提。

```c++
#include <iostream>

class Entity
{
public:
    virtual void PrintName() {} // for generating vTable
};

class Player : public Entity {};

class Enemy : public Entity {};

int main()
{
    Entity* e_player = new Player();
    Entity* e_enemy = new Enemy();

    Player* p1 = (Player*)e_enemy; // allowed but dangerous
    Player* p2 = static_cast<Player*>(e_enemy); // allowed but dangerous

    // 只有在运行时通过RTTI, 才能确认一个Entity类型指针实际指向那个类型
    Player* p3 = dynamic_cast<Player*>(e_enemy); // nullptr
    Player* p4 = dynamic_cast<Player*>(e_player); // legal, p4 != nullptr
}
```

**作用：**虽然有run time性能开销，但`dynamic_cast`可以检测多态情况下的cast是否合法，使代码更鲁棒。

### `const_cast`

将`const`对象cast成non-`const`对象。

```c++
#include <iostream>

class student
{
private:
	int roll;
public:
	// constructor
	student(int r): roll(r) {}

	// A const function that changes roll with the help of const_cast
	void fun() const
	{
		(const_cast<student*>(this))->roll = 5;
	}

	int getRoll() { return roll; }
};
```

若在cast之后尝试修改该对象则属于undefined behavior。

```c++
#include <iostream>

int func1(int* ptr)
{
	return (*ptr + 10);
}

int func2(int* ptr)
{
    *ptr = *ptr + 10;
    return (*ptr);
}

int main()
{
	const int val = 10;
	const int *ptr = &val;

	int *ptr1 = const_cast<int*>(ptr);
	std::cout << func1(ptr1); // 20

    func2(ptr1); // undefined behavior

	return 0;
}
```

与`static_cast`同样提供complie time checking。

```c++
#include <iostream>

int main(void)
{
	int a = 40;
	const int* b = &a;
	char* c = const_cast<char*>(b); // compiler error
	*c = 'A';

	return 0;
}
```

若对象为`const volatile`属性则`volatile`属性也将被一并去除。

### `reinterpret_cast`

`C++` style type puning。
