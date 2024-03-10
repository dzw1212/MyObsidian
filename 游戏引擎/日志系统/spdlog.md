*https://github.com/gabime/spdlog

`spdlog` 是一个高效的 C++ 日志库，旨在提供快速、可扩展和模块化的日志记录功能。它支持控制台、文件、日志轮转和网络日志等多种输出目标。`spdlog` 设计轻量，尽量减少对应用程序性能的影响，同时提供丰富的日志记录功能，包括日志级别、格式化、线程安全等；

# 简单示例

```cpp
#include "spdlog/spdlog.h"
#include "spdlog/sinks/stdout_color_sinks.h"

int main() {
    // 创建控制台日志器
    auto console = spdlog::stdout_color_mt("console");
    // 设置日志级别
    console->set_level(spdlog::level::info);
    // 记录信息
    console->info("Welcome to spdlog!");
    return 0;
}
```
# log级别

```c++
spdlog::warn();
spdlog::critical();
spdlog::info();
spdlog::debug();
spdlog::error();
spdlog::trace();
```

# logger类型

## stdout/stderr

标准日志，输出到控制台，支持彩色高亮；
```c++
auto console = spdlog::stdout_color_mt("console");

spdlog::stdout_logger_mt
spdlog::stderr_color_mt
spdlog::stderr_logger_mt
```

## basic logger

基础日志，用于简单的日志记录需求，将日志输出到文件中；
```c++
#include "spdlog/sinks/basic_file_sink.h"

auto logger = spdlog::basic_logger_mt("basic_logger", "logs/basic.txt");
logger->info("This is a basic logger example.");
```

## rotating logger

循环日志，当超出设置的大小后，会新建log文件，或覆盖之前的log文件；
```c++
#include "spdlog/sinks/rotating_file_sink.h"

//创建3个5mb的日志文件
auto rotating_logger = spdlog::rotating_logger_mt("some_logger_name", "logs/rotating.txt", 1024 * 1024 * 5, 3);
```

## daily log

每日日志，会在每天的某个时刻，新建一个log文件，按日期命名；
```c++
#include "spdlog/sinks/daily_file_sink.h"

//每天2:30时创建日志文件
auto daily_logger = spdlog::daily_logger_mt("daily_logger", "logs/daily.txt", 2, 30);
```

## async log

异步日志，使用单独的线程异步记录日志，以减少对主程序性能的影响；
```c++
#include "spdlog/async.h"
#include "spdlog/sinks/basic_file_sink.h"

auto async_file = spdlog::basic_logger_mt<spdlog::async_factory>("async_file_logger", "logs/async_log.txt");

for (int i = 1; i < 101; ++i)
	async_file->info("Async message #{}", i);
```

## UDP log

UDP日志，将日志从UDP协议发送到指定的服务器和端口；
```c++
#include "spdlog/sinks/udp_sink.h"

spdlog::sinks::udp_sink_config cfg("127.0.0.1", 11091);
auto my_logger = spdlog::udp_logger_mt("udplog", cfg);
my_logger->set_level(spdlog::level::debug);
my_logger->info("hello world");
```

## multi log

多重日志，允许将日志同时输出到多个目的地，比如同时输出到控制台和文件；
```c++
#include "spdlog/sinks/stdout_color_sinks.h"
#include "spdlog/sinks/basic_file_sink.h"

auto console_sink = std::make_shared<spdlog::sinks::stdout_color_sink_mt>();
console_sink->set_level(spdlog::level::warn);
console_sink->set_pattern("[multi_sink_example] [%^%l%$] %v");

auto file_sink = std::make_shared<spdlog::sinks::basic_file_sink_mt>("logs/multisink.txt", true);
file_sink->set_level(spdlog::level::trace);

spdlog::logger logger("multi_sink", { console_sink, file_sink });
logger.set_level(spdlog::level::debug);
logger.warn("this should appear in both console and file");
logger.info("this message should not appear in the console, only in the file");
```

# 设置log屏蔽等级

```c++
spdlog::set_level(spdlog::level::info);
spdlog::debug("This message should not be displayed!");

enum level_enum : int
{
    trace = SPDLOG_LEVEL_TRACE,
    debug = SPDLOG_LEVEL_DEBUG,
    info = SPDLOG_LEVEL_INFO,
    warn = SPDLOG_LEVEL_WARN,
    err = SPDLOG_LEVEL_ERROR,
    critical = SPDLOG_LEVEL_CRITICAL,
    off = SPDLOG_LEVEL_OFF,
    n_levels
};
```

# 设置默认logger

```c++
auto old_logger = spdlog::default_logger();

auto new_logger =spdlog::basic_logger_mt("new_default_logger", "logs/new-default-log.txt", true);
spdlog::set_default_logger(new_logger);
spdlog::set_level(spdlog::level::info);
spdlog::debug("This message should not be displayed!");
spdlog::set_level(spdlog::level::trace);
spdlog::debug("This message should be displayed..");

spdlog::set_default_logger(old_logger);
```

# Backtrace log

```c++
spdlog::enable_backtrace(10); // create ring buffer with capacity of 10  messages
for (int i = 0; i < 100; i++)
{
	spdlog::debug("Backtrace message {}", i); // not logged..
}
spdlog::dump_backtrace(); // log them now!
```

# 自定义日志头格式

```c++
spdlog::set_pattern("*** [%H:%M:%S %z] [thread %t] %v ***");
```

`%Y, %m, %d, %H, %M, %S`：分别代表年、月、日、小时、分钟、秒；
`%e`：毫秒；
`%t`：线程ID；
`%l`：日志级别，使用`%^`和`%$`包裹会显示对应颜色；
`%v`：日志消息本身；
`%s`：源文件名；
`%#`：行号；
`%%`：百分号字符；

---

`%s` 和 `%#` 能够正确显示的前提是，日志消息是通过 spdlog 的宏来记录的，这些宏在预处理阶段自动插入了当前的文件名和行号；

正确的使用方式是使用 `SPDLOG_LOGGER_CALL` 宏或其简化版本（如 `SPDLOG_INFO`, `SPDLOG_ERROR`, `SPDLOG_DEBUG` 等），而不是直接调用日志记录函数（如 info, error, debug 等），这样，spdlog 能够捕获并插入当前的源文件名和行号信息；

```cpp
#include "spdlog/spdlog.h"

int main() {
    spdlog::set_pattern("[%Y-%m-%d %H:%M:%S.%e] [%l] %v (%s:%#)");
    SPDLOG_INFO("This is a test message.");
    return 0;
}
```

# 获取logger

```c++
auto net_logger = std::make_shared<spdlog::logger>("net", daily_sink);
// globally register the loggers so the can be accessed using spdlog::get(logger_name)
spdlog::register_logger(net_logger);

auto same_logger = spdlog::get("net");
```

# 自定义数据类的重载

```c++
#include "spdlog/spdlog.h"
#include "spdlog/fmt/ostr.h" // must be included
#include "spdlog/sinks/stdout_sinks.h"

class some_class {};
std::ostream& operator<<(std::ostream& os, const some_class& c)
{ 
    return os << "some_class"; 
}

//或者通过fmt
class some_class {
    int code;
};

template<typename OStream>
friend OStream &operator<<(OStream &os, const some_class& to_log)
{
    fmt::format_to(std::ostream_iterator<char>(os), "{:04X}", to_log.code);
    return os;
}

void custom_class_example()
{
    some_class c;
    auto console = spdlog::stdout_logger_mt("console");
    console->info("custom class with operator<<: {}..", c);
}
```