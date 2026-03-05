`std::span` 是 C++20 引入的一个非常重要且实用的特性（位于 `<span>` 头文件中）。

简单来说，**`std::span` 是一个轻量级的、不拥有所有权的“视图”（View），用于引用一段连续的内存。**

你可以把它看作是现代 C++ 对传统“指针 + 长度”（`T* ptr, size_t size`）这种组合的**安全且面向对象**的封装。它允许你编写一个函数，既能接受 `std::vector`，又能接受 `std::array`，甚至原生的 C 风格数组，而无需进行重载或模板实例化。


```cpp
#include <iostream>
#include <span>
#include <vector>
#include <array>

// 这个函数可以接受任何连续的 int 内存块
void print_data(std::span<int> data) {
    for (int x : data) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    
    // std::span 可以修改原数据（除非是 span<const int>）
    if (!data.empty()) {
        data[0] = 999; 
    }
}

int main() {
    std::vector<int> vec = {1, 2, 3};
    std::array<int, 3> arr = {4, 5, 6};
    int raw_arr[] = {7, 8, 9};

    // 同一个函数，统一处理
    print_data(vec);      // 自动转换
    print_data(arr);      // 自动转换
    print_data(raw_arr);  // 自动转换

    std::cout << "Vector modified: " << vec[0] << std::endl; // 输出 999
    return 0;
}

```

# 性能优势：零拷贝

* **轻量级**：`std::span` 的内部结构非常简单，通常只包含**一个指针**和**一个大小（size）**。
* **传值（Pass by Value）**：由于它非常小（通常是两个机器字长），在函数传参时，通常直接**按值传递**（`void f(std::span<T> s)`），这比传引用更方便，且没有任何深拷贝开销。

# 高级功能：切片（Slicing）

`std::span` 提供了一个非常强大的 `subspan` 方法，允许你创建原数组“一部分”的视图，而不需要拷贝数据。这类似于 Python 中的切片操作。

```cpp
std::vector<int> vec = {0, 1, 2, 3, 4, 5};
std::span<int> s = vec;

// 取中间一部分：从索引 2 开始，长度为 3
std::span<int> part = s.subspan(2, 3); // 包含 {2, 3, 4}

// 修改 part 会影响原始 vec
part[0] = 100; 
// 此时 vec 变为 {0, 1, 100, 3, 4, 5}

```

# 静态与动态长度

`std::span` 有两个模板参数：

1. **Type (`T`)**: 元素类型。
2. **Extent (`std::size_t`)**: 数组的大小（可选）。

* **动态大小（默认）**：`std::span<int>`。大小在运行时确定，内部存储指针和大小。
* **静态大小**：`std::span<int, 10>`。如果在编译期就知道大小（例如处理定长 `std::array`），通过指定大小，编译器可以优化掉存储“大小”的内存，`span` 内部就只剩一个指针了。

# 悬空引用（Dangling References）

由于 `std::span` **不拥有**内存，你必须确保 `span` 的生命周期短于它所指向的数据的生命周期。

**错误示例：**

```cpp
std::span<int> get_data() {
    std::vector<int> v = {1, 2, 3};
    return v; // ❌ 错误！v 在函数结束时被销毁，返回的 span 指向无效内存。
}

```
