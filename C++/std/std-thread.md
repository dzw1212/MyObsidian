`std::thread` 是 C++11 引入的标准库类，用于创建和管理线程。以下是 `std::thread` 的详细使用方法，包括创建线程、传递参数、使用 lambda 表达式、类成员函数以及线程同步等。

# 创建线程

## 使用无参函数

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    std::thread t(threadFunction); // 创建线程
    t.join(); // 等待线程完成
    return 0;
}
```

## 使用带参函数

```cpp
#include <iostream>
#include <thread>

void threadFunction(int x) {
    std::cout << "Thread function with argument: " << x << std::endl;
}

int main() {
    int arg = 10;
    std::thread t(threadFunction, arg); // 传递参数
    t.join();
    return 0;
}
```

## 使用 lambda 表达式

```cpp
#include <iostream>
#include <thread>

int main() {
    std::thread t([]() {
        std::cout << "Hello from lambda thread!" << std::endl;
    });
    t.join();
    return 0;
}
```

## 使用类成员函数

```cpp
#include <iostream>
#include <thread>

class MyClass {
public:
    void memberFunction(int x) {
        std::cout << "Member function with argument: " << x << std::endl;
    }
};

int main() {
    MyClass obj;
    std::thread t(&MyClass::memberFunction, &obj, 10); // 传递对象指针和参数
    t.join();
    return 0;
}
```

# 线程分离

使用 `detach` 方法将线程分离，使其在后台运行。

```cpp
#include <iostream>
#include <thread>
#include <chrono>

void backgroundTask() {
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Background task completed!" << std::endl;
}

int main() {
    std::thread t(backgroundTask);
    t.detach(); // 分离线程

    std::cout << "Main thread continues..." << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(3)); // 确保后台任务有时间完成
    return 0;
}
```

# 等待线程执行

`join()` 用于等待一个线程完成执行，并清理与该线程相关的资源；

作用：
1. **等待线程完成**：`join()` 会阻塞调用它的线程（通常是主线程），直到被 `join` 的线程完成执行。这意味着主线程会等待子线程执行完毕后再继续执行后续代码；
2. **清理资源**：`join()` 还会清理与线程相关的资源，确保线程对象可以安全销毁。如果不调用 `join()` 或 `detach()`，程序在退出时会抛出异常；

## 判断是否可join

使用 `joinable` 方法检查线程是否可以加入；

1. **防止重复 `join`**：一个线程只能被 `join` 一次。调用 `join` 后，线程对象不再是 `joinable` 的。如果尝试对一个已经被 `join` 的线程再次调用 `join`，会导致程序崩溃；
2. **防止未启动的线程 `join`**：如果一个线程对象没有关联任何可执行的线程（例如，默认构造的 `std::thread` 对象），调用 `join` 也会导致程序崩溃；
3. **防止分离的线程 `join`**：如果一个线程已经被 `detach`，它也不再是 `joinable` 的，调用 `join` 会导致程序崩溃；

```cpp
#include <iostream>
#include <thread>

void threadFunction() {
    std::cout << "Thread function running" << std::endl;
}

int main() {
    std::thread t(threadFunction);

    if (t.joinable()) {
        t.join();
    }

    return 0;
}
```

# 线程同步

使用 `std::mutex` 进行线程同步，防止数据竞争。

```cpp
#include <iostream>
#include <thread>
#include <mutex>

std::mutex mtx;

void printThreadId(int id) {
    std::lock_guard<std::mutex> lock(mtx); // 自动加锁和解锁
    std::cout << "Thread ID: " << id << std::endl;
}

int main() {
    std::thread t1(printThreadId, 1);
    std::thread t2(printThreadId, 2);

    t1.join();
    t2.join();
    return 0;
}
```

```cpp
#include <iostream>
#include <vector>
#include <thread>
#include <string>
#include <cstdlib>
#include <queue>
#include <mutex>
#include <condition_variable>

void convertTgaToDds(const std::string& tgaFile) {
    std::string command = "nvdxt.exe -file " + tgaFile + " -dxt1";
    std::system(command.c_str());
}

class ThreadPool {
public:
    ThreadPool(size_t numThreads);
    ~ThreadPool();
    void enqueue(const std::string& tgaFile);

private:
    std::vector<std::thread> workers;
    std::queue<std::string> tasks;
    std::mutex queueMutex;
    std::condition_variable condition;
    bool stop;

    void worker();
};

ThreadPool::ThreadPool(size_t numThreads) : stop(false) {
    for (size_t i = 0; i < numThreads; ++i) {
        workers.emplace_back(&ThreadPool::worker, this);
    }
}

ThreadPool::~ThreadPool() {
    {
        std::unique_lock<std::mutex> lock(queueMutex);
        stop = true;
    }
    condition.notify_all();
    for (std::thread &worker : workers) {
        worker.join();
    }
}

void ThreadPool::enqueue(const std::string& tgaFile) {
    {
        std::unique_lock<std::mutex> lock(queueMutex);
        tasks.push(tgaFile);
    }
    condition.notify_one();
}

void ThreadPool::worker() {
    while (true) {
        std::string task;
        {
            std::unique_lock<std::mutex> lock(queueMutex);
            condition.wait(lock, [this] { return stop || !tasks.empty(); });
            if (stop && tasks.empty()) {
                return;
            }
            task = tasks.front();
            tasks.pop();
        }
        convertTgaToDds(task);
    }
}

int main() {
    std::vector<std::string> tgaFiles = {
        "file1.tga", "file2.tga", "file3.tga", // 添加更多文件
    };

    size_t numThreads = std::thread::hardware_concurrency();
    if (numThreads == 0) {
        numThreads = 2; // 如果无法获取核心数，默认使用2个线程
    }

    ThreadPool pool(numThreads);

    for (const auto& tgaFile : tgaFiles) {
        pool.enqueue(tgaFile);
    }

    // 线程池析构时会等待所有任务完成
    return 0;
}
```