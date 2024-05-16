`std::async` 是 C++11 引入的一个函数，用于启动一个异步任务，它返回一个与异步任务关联的 `std::future` 对象，通过这个对象可以获取任务的结果。`std::async` 可以让你以并行的方式执行代码，而不需要直接管理线程。

[[std-future]]

# 使用方式

`std::async` 的基本语法如下：

```cpp
auto future = std::async(launch_policy, function, args...);
```

- **launch_policy**（可选）：指定任务的执行策略，可以是 `std::launch::async`、`std::launch::deferred` 或两者的组合。如果省略，则默认为 `std::launch::async | std::launch::deferred`。
  - `std::launch::async`：保证任务会在新线程上异步执行。
  - `std::launch::deferred`：任务会延迟执行，直到调用 `future.get()` 或 `future.wait()` 时才在当前线程上执行。
- **function**：要异步执行的函数。
- **args**：传递给函数的参数。

# 示例

下面是一个使用 `std::async` 的示例，展示如何异步执行一个函数并获取结果：

```cpp
#include <iostream>
#include <future>
#include <chrono>

int compute(int x) {
    // 模拟耗时计算
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return x * x;
}

int main() {
    // 启动异步任务
    auto result = std::async(std::launch::async, compute, 10);

    // 在等待结果的同时可以做其他事情
    std::cout << "Doing other work while waiting for the result..." << std::endl;

    // 获取异步操作的结果
    std::cout << "Computed value: " << result.get() << std::endl;

    return 0;
}
```

在这个例子中：
- `compute` 函数被异步执行，它计算一个数的平方。
- 使用 `std::launch::async` 确保函数在新线程上执行。
- 主线程在等待结果的同时输出了一些文本，然后通过 `result.get()` 获取结果。

# 注意事项

- 调用 `result.get()` 会阻塞当前线程，直到异步任务完成。
- 如果异步任务抛出异常，该异常会在调用 `result.get()` 时被重新抛出。
- 每个 `std::future` 对象只能调用一次 `get()`。如果需要多次访问结果，可以考虑使用 `std::shared_future`。