`std::mutex` 是 C++ 标准库中**用于同步多线程访问共享数据的互斥锁**，它提供了独占的、不可递归的锁定机制，用于保护共享数据免受多个线程同时访问时可能出现的数据竞争和冲突；

使用基本步骤如下：
1. 包含头文件：首先，需要包含 `<mutex>` 头文件以访问 `std::mutex` 类；
2. 创建 `std::mutex` 实例：在共享资源的访问范围内创建一个 `std::mutex` 实例；
3. 锁定和解锁：在访问共享资源之前，使用 `lock()` 方法锁定互斥量。访问完共享资源后，使用 `unlock()` 方法解锁；

为了确保即使在异常情况下也能释放锁，推荐使用 `std::lock_guard` 或 `std::unique_lock` 管理 `std::mutex `的锁定和解锁过程；

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx; // 全局互斥锁
int shared_data = 0; // 共享数据

void increment() {
    mtx.lock(); // 在访问共享数据前锁定
    ++shared_data;
    std::cout << "Data incremented to " << shared_data << std::endl;
    mtx.unlock(); // 完成访问后解锁
}

void decrement() {
    mtx.lock(); // 在访问共享数据前锁定
    --shared_data;
    std::cout << "Data decremented to " << shared_data << std::endl;
    mtx.unlock(); // 完成访问后解锁
}

int main() {
    std::thread t1(increment);
    std::thread t2(decrement);

    t1.join();
    t2.join();

    return 0;
}
```

# std::lock_guard

`std::lock_guard` 是一个作用域锁，它提供了一种便捷的`RAII`（`Resource Acquisition Is Initialization`）风格的方式来管理互斥锁的锁定和解锁操作；
当创建一个 `std::lock_guard` 对象时，它会自动锁定给定的互斥量，并在 `std::lock_guard` 对象的生命周期结束时自动解锁互斥量；这样可以确保即使在发生异常的情况下，互斥量也能被正确地解锁，从而避免死锁；

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx; // 全局互斥锁
int shared_data = 0; // 共享数据

void increment() {
    std::lock_guard<std::mutex> lock(mtx); // 自动锁定
    ++shared_data;
    std::cout << "Data incremented to " << shared_data << std::endl;
    // lock 在这里自动解锁
}

void decrement() {
    std::lock_guard<std::mutex> lock(mtx); // 自动锁定
    --shared_data;
    std::cout << "Data decremented to " << shared_data << std::endl;
    // lock 在这里自动解锁
}

int main() {
    std::thread t1(increment);
    std::thread t2(decrement);

    t1.join();
    t2.join();

    return 0;
}
```

# std::unique_lock

`std::unique_lock` 是 C++ 标准库中的一个类模板，提供了比 `std::lock_guard` 更灵活的互斥锁管理机制；
它允许延迟锁定、尝试锁定、时间锁定以及在作用域结束时自动释放锁。`std::unique_lock` 通常与 `std::mutex` 一起使用，但它比 `std::lock_guard` 更消耗性能，因为它更灵活；

`std::unique_lock` 允许在构造时自动锁定互斥量，也可以选择延迟锁定，可以使用 `lock()`、`unlock()`、`try_lock()` 或 `try_lock_for()`、`try_lock_until()` 方法来控制锁。

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx; // 全局互斥锁

void print_even(int x) {
    if (x % 2 == 0) {
        std::unique_lock<std::mutex> lock(mtx);
        std::cout << x << " is even." << std::endl;
        // `lock` 在这里自动解锁
    } else {
        // 不需要锁
    }
}

void print_odd(int x) {
    if (x % 2 != 0) {
        std::unique_lock<std::mutex> lock(mtx, std::defer_lock); // 延迟锁定
        ...
        lock.lock(); // 显式锁定
        std::cout << x << " is odd." << std::endl;
        // `lock` 在这里自动解锁
    } else {
        // 不需要锁
    }
}

int main() {
    std::thread threads[10];
    for (int i = 0; i < 10; ++i) {
        if (i % 2 == 0) {
            threads[i] = std::thread(print_even, i);
        } else {
            threads[i] = std::thread(print_odd, i);
        }
    }

    for (auto& th : threads) {
        th.join();
    }

    return 0;
}
```

# 双重检查锁定

双重检查锁定（`Double-Checked Locking`，`DCL`）是一种用于减少同步开销的软件设计模式，特别是在实现单例模式或者懒加载时。其核心思想是，在**锁定操作之前先检查资源是否已经被初始化，以避免不必要的锁定开销**；

直接使用 `std::mutex` 可能会引入性能问题，因为每次访问资源时都需要进行锁定和解锁操作，这在资源已经初始化之后实际上是不必要的；

```cpp
#include <mutex>
#include <memory>

std::mutex mtx; // 用于同步的互斥锁
std::unique_ptr<Singleton> instance; // 单例对象的指针

class Singleton {
public:
    static Singleton* getInstance() {
        if (instance == nullptr) { // 第一次检查
            std::lock_guard<std::mutex> lock(mtx);
            if (instance == nullptr) { // 第二次检查，防止在加锁的过程中被其他线程初始化
                instance = std::make_unique<Singleton>();
            }
        }
        return instance.get();
    }

private:
    Singleton() {} // 私有构造函数
    // 禁止拷贝和赋值
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;
};
```

需要注意的是，由于 C++11 引入的内存模型保证了原子操作的顺序性，上述代码在现代C++环境中通常是安全的。但在某些情况下，或在老版本的C++标准中，双重检查锁定可能会因为编译器优化或内存模型的差异而不安全；

在C++11及以后，推荐使用 `std::call_once` 和 `std::once_flag` 来安全且高效地实现单例模式或懒加载；
1. 代码更简单；
2. 避免了双重检查锁定的潜在编译器优化问题；
3. 性能相比上锁解锁更佳；
4. 异常安全：如果初始化函数抛出异常，`std::call_once`能保证异常被传播出去，有机会处理异常并重试初始化；

[[std-call_once]]

