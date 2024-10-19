

C++17 引入了执行策略（`Execution Policies`），用于指定标准库算法的执行方式。这些策略允许程序员控制算法的并行性和顺序性，从而提高性能。以下是 C++ 中的三种主要执行策略：

# 顺序 seq

- **描述**：顺序执行策略。
- **行为**：算法按照元素的顺序逐个执行，类似于传统的单线程执行。
- **适用场景**：当算法需要严格的顺序执行，或者在单线程环境中运行时使用。

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data = {1, 2, 3, 4, 5};
std::for_each(std::execution::seq, data.begin(), data.end(), [](int& n) {
    n *= 2;
});
```

# 并行 par

- **描述**：并行执行策略。
- **行为**：算法可以在多个线程上并行执行，但仍然保证每个线程内的顺序。
- **适用场景**：适用于不依赖于元素处理顺序的计算密集型任务，且需要利用多核处理器的并行能力。

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data = {1, 2, 3, 4, 5};
std::for_each(std::execution::par, data.begin(), data.end(), [](int& n) {
    n *= 2;
});
```

# 无序 par_unseq

- **描述**：并行和无序执行策略。
- **行为**：算法可以在多个线程上并行执行，并且不保证元素的处理顺序。
- **适用场景**：适用于不依赖于元素处理顺序的计算密集型任务，且需要最大化并行性能。

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data = {1, 2, 3, 4, 5};
std::for_each(std::execution::par_unseq, data.begin(), data.end(), [](int& n) {
    n *= 2;
});
```

---

- **线程安全**：当使用 `par` 或 `par_unseq` 时，确保操作是线程安全的。
- **硬件支持**：并行执行策略的性能提升依赖于硬件支持（如多核 CPU）。
- **编译器支持**：确保你的编译器支持 C++17 及以上版本，并启用了并行算法支持。

# 配合的算法

除了`std::for_each`外，执行策略还可以配合以下的一些算法：

## 1. `std::transform`

用于将一个范围内的元素转换为另一个范围。

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data = {1, 2, 3, 4, 5};
std::vector<int> result(data.size());

std::transform(std::execution::par, data.begin(), data.end(), result.begin(), [](int n) {
    return n * n;
});
```

## 2. `std::sort`

用于对范围内的元素进行排序。

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data = {5, 3, 1, 4, 2};

std::sort(std::execution::par, data.begin(), data.end());
```

## 3. `std::reduce`

用于对范围内的元素进行归约操作（类似于 `std::accumulate`，但支持并行）。

```cpp
#include <vector>
#include <numeric>
#include <execution>

std::vector<int> data = {1, 2, 3, 4, 5};

int sum = std::reduce(std::execution::par, data.begin(), data.end(), 0);
```

## 4. `std::copy`

用于将一个范围内的元素复制到另一个范围。

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data = {1, 2, 3, 4, 5};
std::vector<int> result(data.size());

std::copy(std::execution::par, data.begin(), data.end(), result.begin());
```

## 5. `std::fill`

用于将一个范围内的元素填充为指定的值。

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data(5);

std::fill(std::execution::par, data.begin(), data.end(), 42);
```

## 6. `std::generate`

用于根据生成器函数生成范围内的元素。

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data(5);

std::generate(std::execution::par, data.begin(), data.end(), [n = 0]() mutable { return n++; });
```

## 7. `std::unique`

用于移除范围内的重复元素。

```cpp
#include <vector>
#include <algorithm>
#include <execution>

std::vector<int> data = {1, 2, 2, 3, 4, 4, 5};

auto end = std::unique(std::execution::par, data.begin(), data.end());
data.erase(end, data.end());
```
