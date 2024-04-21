工厂设计模式（`Factory Design Pattern`）用于创建对象而不必暴露创建逻辑给客户端，并且通过使用一个共同的接口来指向新创建的对象；

```cpp
#include <iostream>
#include <memory>

// Product基类
class Product {
public:
    virtual ~Product() {}
    virtual void Operation() const = 0;
};

// ConcreteProductA类
class ConcreteProductA : public Product {
public:
    void Operation() const override {
        std::cout << "Operation of ConcreteProductA\n";
    }
};

// ConcreteProductB类
class ConcreteProductB : public Product {
public:
    void Operation() const override {
        std::cout << "Operation of ConcreteProductB\n";
    }
};

// Factory类
class Factory {
public:
    enum PRODUCT_TYPE {
        PRODUCT_A,
        PRODUCT_B
    };

    std::unique_ptr<Product> CreateProduct(PRODUCT_TYPE type) {
        switch (type) {
            case PRODUCT_A: return std::make_unique<ConcreteProductA>();
            case PRODUCT_B: return std::make_unique<ConcreteProductB>();
            default: return nullptr;
        }
    }
};
```

客户端代码（在这个例子中是 `main` 函数）不需要知道具体的产品类，只需要知道 `Product` 接口和工厂类即可。这样，当引入新的产品类时，只需要修改工厂类而不需要修改客户端代码，从而提高了系统的可扩展性。

```cpp
int main() {
    Factory factory;
    auto productA = factory.CreateProduct(Factory::PRODUCT_A);
    if (productA) {
        productA->Operation();
    }

    auto productB = factory.CreateProduct(Factory::PRODUCT_B);
    if (productB) {
        productB->Operation();
    }

    return 0;
}
```

# 派生类构造函数参数不同

```cpp
#include <iostream>
#include <memory>

// 基类
class Base {
public:
    virtual ~Base() = default;
    virtual void display() const = 0;
};

// 派生类A，有一个参数
class A : public Base {
    int x;
public:
    A(int x) : x(x) {}
    void display() const override {
        std::cout << "A: " << x << std::endl;
    }
};

// 派生类B，没有参数
class B : public Base {
public:
    B() {}
    void display() const override {
        std::cout << "B" << std::endl;
    }
};

// 派生类C，有三个参数
class C : public Base {
    int x, y, z;
public:
    C(int x, int y, int z) : x(x), y(y), z(z) {}
    void display() const override {
        std::cout << "C: " << x << ", " << y << ", " << z << std::endl;
    }
};

// 工厂类
class Factory {
public:
    template<typename T, typename... Args>
    static std::unique_ptr<Base> create(Args&&... args) {
        return std::make_unique<T>(std::forward<Args>(args)...);
    }
};

int main() {
    auto a = Factory::create<A>(10);
    auto b = Factory::create<B>();
    auto c = Factory::create<C>(1, 2, 3);

    a->display();
    b->display();
    c->display();

    return 0;
}
```