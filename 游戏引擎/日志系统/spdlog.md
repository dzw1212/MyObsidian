*https://github.com/gabime/spdlog

## log级别

```c++
spdlog::warn();
spdlog::critical();
spdlog::info();
spdlog::debug();
```

## logger类型

### stdout/stderr

```c++
auto console = spdlog::stdout_color_mt("console");

spdlog::stdout_logger_mt
spdlog::stderr_color_mt
spdlog::stderr_logger_mt
```

### basic logger

```c++
auto my_logger = spdlog::basic_logger_mt("file_logger", "logs/basic-log.txt", true);
```

### rotating logger

当超出设置的大小后，会新建log文件，或覆盖之前的log文件；
```c++
// Create a file rotating logger with 5mb size max and 3 rotated files.
auto rotating_logger = spdlog::rotating_logger_mt("some_logger_name", "logs/rotating.txt", 1048576 * 5, 3);
```

### daily log

会在每天的某个时刻，新建一个log文件；
```c++
// Create a daily logger - a new file is created every day on 2:30am.
auto daily_logger = spdlog::daily_logger_mt("daily_logger", "logs/daily.txt", 2, 30);
```

### async log

```c++
auto async_file = spdlog::basic_logger_mt<spdlog::async_factory>("async_file_logger", "logs/async_log.txt");

for (int i = 1; i < 101; ++i)
{
	async_file->info("Async message #{}", i);
}
```

### UDP log

```c++
spdlog::sinks::udp_sink_config cfg("127.0.0.1", 11091);
auto my_logger = spdlog::udp_logger_mt("udplog", cfg);
my_logger->set_level(spdlog::level::debug);
my_logger->info("hello world");
```

### multi log

能够同时在stdout和多个文件上打印log；
```c++
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



## 设置log屏蔽等级

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

## 设置默认logger

```c++
auto old_logger = spdlog::default_logger();

auto new_logger = spdlog::basic_logger_mt("new_default_logger", "logs/new-default-log.txt", true);
spdlog::set_default_logger(new_logger);
spdlog::set_level(spdlog::level::info);
spdlog::debug("This message should not be displayed!");
spdlog::set_level(spdlog::level::trace);
spdlog::debug("This message should be displayed..");

spdlog::set_default_logger(old_logger);
```



## Backtrace log

```c++
spdlog::enable_backtrace(10); // create ring buffer with capacity of 10  messages
for (int i = 0; i < 100; i++)
{
	spdlog::debug("Backtrace message {}", i); // not logged..
}
spdlog::dump_backtrace(); // log them now!
```

## 自定义日志头格式

```c++
spdlog::set_pattern("*** [%H:%M:%S %z] [thread %t] %v ***");
```

## 获取logger

```c++
auto net_logger = std::make_shared<spdlog::logger>("net", daily_sink);
// globally register the loggers so the can be accessed using spdlog::get(logger_name)
spdlog::register_logger(net_logger);

auto same_logger = spdlog::get("net");
```

## 自定义数据类的重载

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