*https://github.com/tanakh/cmdline*

`takah/cmdline` 是一个轻量级的C++库，用于解析命令行参数。这个库的设计目标是简单易用，适合小型项目或需要快速实现命令行参数解析的场景。以下是对`takah/cmdline`库的一些介绍：

# 特点

1. **单头文件**：
   - `takah/cmdline`是一个单头文件库，通常只需要包含`cmdline.h`即可使用。这使得它非常易于集成到现有项目中。

2. **简单易用**：
   - 该库提供了简单的接口来定义和解析命令行参数。用户可以快速上手，无需复杂的配置。

3. **支持多种参数类型**：
   - 支持基本的数据类型，如`int`、`float`、`string`等，并且可以通过模板机制扩展支持自定义类型。

4. **自动生成帮助信息**：
   - 可以自动生成命令行帮助信息，帮助用户了解可用的命令行选项及其用法。

# 基本用法

以下是一个使用`takah/cmdline`库的简单示例：

```cpp
#include "cmdline.h"

int main(int argc, char* argv[]) {
    cmdline::parser parser;
    parser.add<std::string>("host", 'h', "host name", true, "localhost");
    parser.add<int>("port", 'p', "port number", false, 80, cmdline::range(1, 65535));
    parser.add("help", 0, "print this message");

    parser.parse_check(argc, argv);

    if (parser.exist("help")) {
        std::cout << parser.usage();
        return 0;
    }

    std::string host = parser.get<std::string>("host");
    int port = parser.get<int>("port");

    std::cout << "Connecting to " << host << " on port " << port << std::endl;

    return 0;
}
```

