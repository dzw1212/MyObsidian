`std::future` 是 C++ 标准库中的一个模板类，用于表示异步操作的结果。它提供了一种访问异步操作结果的机制，无论这些操作是由 `std::async`、`std::packaged_task` 或 `std::promise` 所启动的。`std::future` 通常用于从另一个线程或异步任务中获取结果，而不会阻塞当前线程，直到结果准备好为止。

[[std-async]]

# 主要功能

- **获取异步结果**：`std::future` 提供了 `get()` 方法，该方法会阻塞调用线程，直到异步操作完成并返回结果。一旦调用了 `get()`，`future` 对象就会变得无效，你不能再次从同一个 `future` 对象中获取结果。
- **检查状态**：可以使用 `wait()` 方法等待 `future` 对象关联的异步操作完成，而不获取结果。此外，`wait_for()` 和 `wait_until()` 方法允许带有超时的等待，让你可以在指定时间后继续执行，而不管结果是否已经准备好。
- **查询就绪状态**：`valid()` 方法用于检查 `future` 对象是否有一个有效的异步状态与之关联，即它是否代表了某个未来的结果。

# 使用场景

`std::future` 适用于以下情况：
- 当你需要从另一个线程或异步操作中获取数据或结果时。
- 当你希望在等待数据或结果时，当前线程可以继续执行其他任务，直到需要结果数据时再进行同步。
- `std::future` 本身并不提供直接的机制来通知当前线程异步调用已完成。

# 示例代码

下面是一个使用 `std::future` 来获取异步操作结果的简单示例：

```cpp
#include <iostream>
#include <future>
#include <thread>

int performComputation(int x) {
    // 模拟耗时操作
    std::this_thread::sleep_for(std::chrono::seconds(2));
    return x * x;
}

int main() {
    // 启动异步任务
    std::future<int> result = std::async(std::launch::async, performComputation, 10);

    // 在等待结果的同时可以做其他事情
    std::cout << "Waiting for the result..." << std::endl;

    // 获取异步操作的结果
    std::cout << "Result is: " << result.get() << std::endl;

    return 0;
}
```

在这个例子中，`std::async` 用于启动一个异步任务，该任务计算一个数的平方，并返回结果。主线程在等待结果的同时可以继续执行，最后通过 `result.get()` 获取异步操作的结果。