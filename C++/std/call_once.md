`std::call_once` 是 C++11 引入的一个功能，它**确保一个给定的函数只被调用一次，即使在面对并发执行时也是如此**。这对于延迟初始化和单例模式等场景特别有用，因为它们通常只需要初始化一次；

`std::call_once` 需要与 `std::once_flag` 一起使用，`std::once_flag` 是一个没有公共构造函数的类，用于保证某个函数只被执行一次，每个 `std::once_flag` 对象与一个特定的函数调用相关联；在程序的整个生命周期中只调用一次；

```cpp
#include <iostream>
#include <mutex>
#include <thread>

std::once_flag flag;

void do_once() {
    std::call_once(flag, [](){ std::cout << "Called once" << std::endl; });
}

int main() {
    std::thread t1(do_once);
    std::thread t2(do_once);
    std::thread t3(do_once);

    t1.join();
    t2.join();
    t3.join();

    return 0;
}
```