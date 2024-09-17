
Q：选择多进程还是多线程？
A：
- I/O密集型任务（如网络请求、文件读写）：多线程更合适；
- CPU密集型任务（如计算密集型操作）：多进程更合适；

Q：多进程的进程数量如何确定？
A：通常，进程数量可以设置为等于或略多于CPU核心数，以充分利用多核CPU的计算能力；
	在C++中，可以使用 `std::thread::hardware_concurrency()` 获取系统可以并行执行的硬件线程数（即逻辑处理器数）；


---

# 接口

## Windows


## Unix

```cpp
fork();
```
# 实例

```cpp
//Unix/Linux
#include <iostream>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    const int num_processes = 10;
    pid_t pids[num_processes];

    for (int i = 0; i < num_processes; ++i) {
        pids[i] = fork();

        if (pids[i] < 0) {
            // fork失败
            std::cerr << "Fork failed for process " << i << std::endl;
            return 1;
        } else if (pids[i] == 0) {
            // 子进程
            std::cout << "This is child process " << i << ". PID: " << getpid() << std::endl;
            // 子进程的代码
            return 0; // 子进程完成后退出
        }
    }

    // 父进程等待所有子进程结束
    for (int i = 0; i < num_processes; ++i) {
        waitpid(pids[i], NULL, 0);
    }

    std::cout << "All child processes have finished." << std::endl;
    return 0;
}
```

```cpp
//windows
#include <windows.h>
#include <iostream>

int main() {
    const int num_processes = 10;
    PROCESS_INFORMATION processInfo[num_processes];
    STARTUPINFO startupInfo[num_processes];

    for (int i = 0; i < num_processes; ++i) {
        ZeroMemory(&startupInfo[i], sizeof(startupInfo[i]));
        startupInfo[i].cb = sizeof(startupInfo[i]);
        ZeroMemory(&processInfo[i], sizeof(processInfo[i]));

        // 创建子进程
        if (!CreateProcess(
                NULL,                   // 应用程序名称
                (LPSTR)"child.exe",     // 命令行参数
                NULL,                   // 进程安全属性
                NULL,                   // 线程安全属性
                FALSE,                  // 是否继承句柄
                0,                      // 创建标志
                NULL,                   // 环境变量
                NULL,                   // 当前目录
                &startupInfo[i],        // 启动信息
                &processInfo[i])) {     // 进程信息
            std::cerr << "CreateProcess failed for process " << i << std::endl;
            return 1;
        } else {
            std::cout << "Created child process " << i << ". PID: " << processInfo[i].dwProcessId << std::endl;
        }
    }

    // 等待所有子进程结束
    for (int i = 0; i < num_processes; ++i) {
        WaitForSingleObject(processInfo[i].hProcess, INFINITE);
        CloseHandle(processInfo[i].hProcess);
        CloseHandle(processInfo[i].hThread);
    }

    std::cout << "All child processes have finished." << std::endl;
    return 0;
}
```