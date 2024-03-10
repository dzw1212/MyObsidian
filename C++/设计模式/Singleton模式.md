单例模式（`Singleton Pattern`）用于确保一个类只有一个实例，并提供一个全局访问点；

在C++中实现单例模式通常涉及以下几个步骤：
1. 将构造函数、复制构造函数和赋值运算符设置为私有或删除，以防止外部通过这些方式创建类的实例；
2. 在类内部创建一个静态私有实例指针；
3. 提供一个公共的静态方法（通常命名为`getInstance`），用于获取这个唯一的实例。如果实例不存在，该方法会创建一个实例并返回；

```cpp
class Singleton {
private:
    // 私有静态指针，指向类的唯一实例
    static Singleton* instance;

    // 私有构造函数，防止外部通过new创建实例
    Singleton() {}

    // 禁止复制构造函数和赋值运算符
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

public:
    // 公共静态方法，用于获取类的唯一实例
    static Singleton* getInstance() {
        if (instance == nullptr) {
            instance = new Singleton();
        }
        return instance;
    }
};

// 初始化静态成员变量
Singleton* Singleton::instance = nullptr;
```

多线程安全版本（`mutex`）：
[[mutex]]
```cpp
#include <mutex>

class Singleton {
private:
    static Singleton* instance;
    static std::mutex mutex;

    // 私有构造函数
    Singleton() {}

    // 禁止复制构造函数和赋值运算符
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

public:
    // 使用双重检查锁定模式确保线程安全
    static Singleton* getInstance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mutex); // 上锁
            if (instance == nullptr) {
                instance = new Singleton();
            }
        }
        return instance;
    }
};

// 初始化静态成员变量
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mutex;
```

使用智能指针版本：
```cpp
#include <mutex>
#include <memory>

class Singleton {
private:
    static std::unique_ptr<Singleton> instance;
    static std::mutex mutex;

    // 私有构造函数
    Singleton() {}

    // 禁止复制构造函数和赋值运算符
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

public:
    // 使用双重检查锁定模式确保线程安全
    static Singleton* getInstance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mutex); // 上锁
            if (instance == nullptr) {
                instance = std::unique_ptr<Singleton>(new Singleton());
            }
        }
        return instance.get();
    }
};

// 初始化静态成员变量
std::unique_ptr<Singleton> Singleton::instance = nullptr;
std::mutex Singleton::mutex;
```

多线程安全版本（`call_once`，C++11后可用）：
[[call_once]]
```cpp
#include <iostream>
#include <memory>
#include <mutex>

class Singleton {
public:
    static Singleton* getInstance() {
        std::call_once(initInstanceFlag, &Singleton::initSingleton);
        return instance.get();
    }

private:
    Singleton() {} // 私有构造函数
    ~Singleton() {} // 私有析构函数
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    static void initSingleton() {
        instance.reset(new Singleton());
    }

    static std::unique_ptr<Singleton> instance;
    static std::once_flag initInstanceFlag;
};

// 静态成员初始化
std::unique_ptr<Singleton> Singleton::instance;
std::once_flag Singleton::initInstanceFlag;
```
