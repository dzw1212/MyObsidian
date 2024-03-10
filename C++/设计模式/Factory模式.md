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