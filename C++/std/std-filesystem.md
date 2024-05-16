`std::filesystem` 是 C++17 引入的一个库，用于操作文件系统，如创建、删除文件或目录，查询文件属性等。以下是一些基本用法示例：

### 包含头文件
要使用 `std::filesystem`，首先需要包含相应的头文件：
```cpp
#include <filesystem>
namespace fs = std::filesystem;
```

### 创建目录
创建一个新目录：
```cpp
fs::create_directory("example_dir");
```

### 删除目录
删除一个目录（如果目录为空）：
```cpp
fs::remove("example_dir");
```
要删除非空目录及其内容，可以使用 `fs::remove_all`。

### 检查文件或目录是否存在
```cpp
bool exists = fs::exists("example_file.txt");
```

### 获取文件大小
```cpp
auto size = fs::file_size("example_file.txt");
```

### 遍历目录
遍历一个目录中的所有文件和子目录：
```cpp
for(auto& p: fs::directory_iterator("example_dir"))
    std::cout << p.path() << std::endl;
```

### 复制文件
```cpp
fs::copy_file("source.txt", "destination.txt");
```

### 移动或重命名文件
```cpp
fs::rename("old_name.txt", "new_name.txt");
```

### 获取文件或目录的最后修改时间
```cpp
auto ftime = fs::last_write_time("example_file.txt");
std::time_t cftime = decltype(ftime)::clock::to_time_t(ftime); // 转换为 std::time_t
```

### 修改文件或目录的最后修改时间
```cpp
fs::file_time_type newTime = /* 指定时间 */;
fs::last_write_time("example_file.txt", newTime);
```

这些是 `std::filesystem` 的一些基本用法。这个库提供了丰富的功能来处理文件系统相关的任务，可以极大地简化文件和目录的操作。