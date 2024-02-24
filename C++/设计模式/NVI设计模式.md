`Non-Virtual Interface` (`NVI`) 模式是一种设计模式，用于在基类中定义一个公共的非虚拟接口，然后在派生类中实现具体的行为。这种模式通常用于确保在执行派生类特定的行为之前或之后执行一些共通的代码，从而减少代码重复并提高代码的可维护性；

- 基类中：定义一个`private`或`protected`的虚函数（为了不让派生类直接调用，使用`protected`用的是正常的虚函数机制，而使用`private`用的是同名遮蔽机制），再定义一个公开的函数作为调用虚函数的接口（也称为外覆器`Wrapper`），公开函数中可以在虚函数之前或之后做准备或善后工作；

- 派生类：重写基类的虚函数，然后调用基类中定义的公开函数；

总而言之，`NVI`设计定制了接口的使用方式；

```cpp
#include <iostream>

class Base {
public:
    // 公共的非虚拟接口
    void publicInterface() const {
        // 在调用特定实现之前可以执行一些操作
        std::cout << "Common pre-processing steps\n";
        
        // 调用具体实现
        implementation();
        
        // 在调用特定实现之后可以执行一些操作
        std::cout << "Common post-processing steps\n";
    }

protected:
    // 为派生类提供的虚拟接口
    virtual void implementation() const = 0;
};

class Derived : public Base {
protected:
    // 派生类的具体实现
    void implementation() const override {
        std::cout << "Derived implementation\n";
    }
};

int main() {
    Derived d;
    d.publicInterface(); // 调用公共接口
}
```