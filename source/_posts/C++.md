title: C++
date: 2023-11-22 21:12:56
tags:
---
### using

#### using声明

一条 `using 声明` 语句一次只引入命名空间的一个成员。它使得我们可以清楚知道程序中所引用的到底是哪个名字。如：

```
using namespace_name::name;
```

#### 构造函数的using声明

在 C++11 中，派生类能够重用其直接基类定义的构造函数。

```
class Derived : Base {
public:
    using Base::Base;
    /* ... */
};
```

如上 using 声明，对于基类的每个构造函数，编译器都生成一个与之对应（形参列表完全相同）的派生类构造函数。生成如下类型构造函数：

```
Derived(parms) : Base(args) { }
```

我在这里解释一下派生类：在C++中，派生类是从另一个类（称为基类或父类）继承属性和行为的类。派生类可以使用基类中定义的成员变量和成员函数，并可以添加自己的成员变量和成员函数。这种继承关系允许派生类重用基类的代码，并在需要时进行扩展或修改。

```cpp
#include <iostream>

// 定义基类
class Shape {
public:
    void setWidth(int w) {
        width = w;
    }

    void setHeight(int h) {
        height = h;
    }

protected:
    int width;
    int height;
};

// 定义派生类
class Rectangle : public Shape {
public:
    int getArea() {
        return width * height;
    }
};

int main() {
    Rectangle rect;

    rect.setWidth(5);
    rect.setHeight(10);

    int area = rect.getArea();
    std::cout << "矩形的面积为：" << area << std::endl;

    return 0;
}
```

通过派生类的继承，我们可以在不影响基类的情况下，对类进行修改和扩展，实现更高级的功能。
#### using指示

`using 指示` 使得某个特定命名空间中所有名字都可见，这样我们就无需再为它们添加任何前缀限定符了。如：

```
using namespace_name name;
```

#### 尽量少使用using指示污染命名空间
***
一般说来，使用 using 命令比使用 using 编译命令更安全，这是由于它**只导入了指定的名称**。如果该名称与局部名称发生冲突，编译器将**发出指示**。using编译命令导入所有的名称，包括可能并不需要的名称。如果与局部名称发生冲突，则**局部名称将覆盖名称空间版本**，而编译器**并不会发出警告**。另外，名称空间的开放性意味着名称空间的名称可能分散在多个地方，这使得难以准确知道添加了哪些名称。
***
应该多使用 `using 声明`

```
int x;
std::cin >> x ;
std::cout << x << std::endl;
```

或者

```
using std::cin;
using std::cout;
using std::endl;
int x;
cin >> x;
cout << x << endl;
```

### :: 范围解析运算符

#### 分类

1. 全局作用域符（`::name`）：用于类型名称（类、类成员、成员函数、变量等）前，表示作用域为全局命名空间
2. 类作用域符（`class::name`）：用于表示指定类型的作用域范围是具体某个类的
3. 命名空间作用域符（`namespace::name`）:用于表示指定类型的作用域范围是具体某个命名空间的

:: 使用

```
int count = 11;         // 全局（::）的 count

class A {
public:
    static int count;   // 类 A 的 count（A::count）
};
int A::count = 21;

void fun()
{
    int count = 31;     // 初始化局部的 count 为 31
    count = 32;         // 设置局部的 count 的值为 32
}

int main() {
    ::count = 12;       // 测试 1：设置全局的 count 的值为 12

    A::count = 22;      // 测试 2：设置类 A 的 count 为 22

    fun();                // 测试 3

    return 0;
}
```


### enum 枚举类型

#### 限定作用域的枚举类型

```
enum class open_modes { input, output, append };
```

#### 不限定作用域的枚举类型

与传统的作用域限定枚举类型（Scoped enumeration）相比，它的枚举值的作用域不受限制，可以直接使用枚举值而无需通过枚举类型进行限定。

```
enum color { red, yellow, green };
enum { floatPrec = 6, doublePrec = 10 };
```

### C++的虚函数到底有什么用

C++中的虚函数是面向对象编程中的重要概念之一，它用于实现多态性。通过使用虚函数，可以在基类和派生类之间建立动态绑定关系，使得程序能够根据对象的实际类型来调用相应的函数。

以下是一些使用虚函数的示例，以说明其用途和好处：

1. 多态性（Polymorphism）：虚函数允许在基类中声明一个函数，并在派生类中进行重写。当通过基类指针或引用调用这个函数时，实际调用的是派生类中的版本。这种行为称为动态绑定，它实现了多态性。


```cpp
class Shape {
public:
    virtual void Draw() {
        std::cout << "Drawing a shape." << std::endl;
    }
};

class Circle : public Shape {
public:
    void Draw() override {
        std::cout << "Drawing a circle." << std::endl;
    }
};

int main() {
    Shape* shape = new Circle();
    shape->Draw();  // 输出 "Drawing a circle."
    delete shape;
    return 0;
}
```

在上面的示例中，`Shape` 是一个基类，其中的 `Draw` 函数被声明为虚函数。`Circle` 是 `Shape` 的派生类，并重写了 `Draw` 函数。在 `main` 函数中，将 `Circle` 对象的地址赋值给 `Shape` 类型的指针 `shape`。当调用 `shape->Draw()` 时，实际上调用的是 `Circle` 类中重写的 `Draw` 函数。这就展示了多态性的特性，即通过基类指针调用派生类的函数。

2. 实现基类的通用接口：虚函数允许在基类中定义一个通用的接口，而将实际的实现留给派生类。这样可以实现对不同类型的对象进行统一的操作。

```cpp
class Animal {
public:
    virtual void MakeSound() = 0;  // 纯虚函数
};

class Dog : public Animal {
public:
    void MakeSound() override {
        std::cout << "Dog: Woof!" << std::endl;
    }
};

class Cat : public Animal {
public:
    void MakeSound() override {
        std::cout << "Cat: Meow!" << std::endl;
    }
};

int main() {
    Animal* animal1 = new Dog();
    Animal* animal2 = new Cat();

    animal1->MakeSound();  // 输出 "Dog: Woof!"
    animal2->MakeSound();  // 输出 "Cat: Meow!"

    delete animal1;
    delete animal2;
    return 0;
}
```

在上面的示例中，`Animal` 是一个抽象基类，其中的 `MakeSound` 函数声明为纯虚函数。`Dog` 和 `Cat` 是 `Animal` 的派生类，并分别提供了它们自己的实现。在 `main` 函数中，通过基类指针调用 `MakeSound` 函数，可以对不同类型的动物对象进行统一的操作。
