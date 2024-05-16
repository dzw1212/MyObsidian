`std::variant`是 C++17 引入的一个类型安全的联合体，它可以存储其模板参数中指定的任何一种类型的值。与传统的联合体（`union`）相比，`std::variant` 在类型安全性、易用性上有显著的改进。它确保了只有定义的类型可以被存储，并且在任何时候都知道当前存储的是哪种类型的值；

- `std::get`：你可以使用 `std::get<T>(v)` 来获取 `std::variant` 中存储的类型为 `T` 的值。如果当前存储的类型不是 `T`，则会抛出 `std::bad_variant_access` 异常；

- `std::get_if`：为了避免异常，你可以使用 `std::get_if<T>(&v)`，它会返回一个指向存储的值的指针，如果类型不匹配，则返回 `nullptr`；

- `std::visit`：`std::visit` 允许你对 `std::variant` 中存储的值执行操作，而不必事先知道存储的是哪种类型。它接受一个可调用对象和一个 `std::variant` 实例，根据 `std::variant` 当前存储的类型调用可调用对象的相应重载；


```cpp
#include <variant>
#include <iostream>
#include <string>

int main() {
    // 定义一个可以存储 int, double, 和 std::string 的 variant
    std::variant<int, double, std::string> v;

    v = 20; // 存储 int
    std::cout << std::get<int>(v) << std::endl;

    v = 3.14; // 存储 double
    std::cout << std::get<double>(v) << std::endl;

    v = "Hello, std::variant!"; // 存储 std::string
    std::cout << std::get<std::string>(v) << std::endl;

    return 0;
}
```

```cpp
#include <variant>
#include <iostream>

struct Printer {
    void operator()(int i) const { std::cout << "int: " << i << std::endl; }
    void operator()(double f) const { std::cout << "double: " << f << std::endl; }
    void operator()(const std::string& s) const { std::cout << "string: " << s << std::endl; }
};

int main() {
    std::variant<int, double, std::string> v = 1;
    std::visit(Printer{}, v); // 输出: int: 1

    v = 3.14;
    std::visit(Printer{}, v); // 输出: double: 3.14

    v = "variant";
    std::visit(Printer{}, v); // 输出: string: variant

    return 0;
}
```