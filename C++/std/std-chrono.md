`std::chrono` 是 C++11 引入的一个库，用于处理日期和时间。这个库提供了时间点（`time_point`）、持续时间（`duration`）和时钟（`clock`）的概念，使得时间相关的操作更加直观和安全；


- **持续时间（Duration）**：表示时间的长度。例如，10秒可以表示为 `std::chrono::seconds(10)`；
- **时间点（Time Point）**：表示某个特定的时间点。例如，现在的时间可以用 `std::chrono::system_clock::now()` 表示；
- **时钟（Clock）**：提供访问当前时间点的方法。常用的时钟有 `std::chrono::system_clock`（系统时钟，通常用于表示现实世界的时间）和 `std::chrono::steady_clock`（稳定时钟，主要用于测量时间间隔）；

# 时间段 duration

```c++
template <class Rep, class Period = ratio<1> > class duration;

//Rep为数值类型，如int,float,double
//Period表示“用秒定义的时间单位”

template <intmax_t N, intmax_t D = 1> class ratio;

//std::ratio表示一个分数值，N为分子(numerator)，D为分母(denominator)
```

| 已定义变量              | 实际含义                              |
| :----------------- | :-------------------------------- |
| std::chrono::hours | `std::chrono::duration<int, 3600> |
| minutes            | `duration<int, std::ratio<60>>    |
| seconds            | `duration<long long>              |
| milliseconds       | `duration<long long, std::milli>  |
| nanoseconds        | `duration<long long, std::nano>   |

```c++
std::milli // 1 / 10^3
std::micro // 1 / 10^6
std::nano  // 1 / 10^9
```

也可以自定义时间段：
```c++
using day_duration = std::chrono::duration<int, ratio<3600*24>>;
```

成员函数`count()`返回Rep类型的Period数量：
```c++
day_duration Days(10);
std::cout << "day count = " << Days.count() << std::endl; //10


std::chrono::hours hours(1);
std::cout << "hour count = " << hours.count()<< std::endl; //1
```

获取时间段对应的秒数：
```c++
std::chrono::minutes::period::num //60
std::chrono::minutes::period::den //1
```

使用`duration_cast`进行时间段的转换：
```c++
std::chrono::hours hours(1); //1个小时=60分钟
auto mins = std::chrono::duration_cast<std::chrono::minutes>(hours);
std::cout << "min count = " << mins.count() << std::endl; //60
```

# 时间点 time_point

```c++
template <class Clock, class Duration = typename Clock::duration>  class time_point;
```

获取当前时间点与对应时间戳：
```c++
auto tp_now = std::chrono::system_clock::now();
std::cout << tp_now.time_since_epoch().count() << std::endl;
```

使用`time_point_cast`进行时间点的转换：
```c++
auto tp_now = std::chrono::system_clock::now();
std::cout << tp_now.time_since_epoch().count() << std::endl;
//16741054848268017

auto tp_now_sec = std::chrono::time_point_cast<std::chrono::seconds>(tp_now);
std::cout << tp_now_sec.time_since_epoch().count() << std::endl;
//1674105484

auto tp_now_millisec = std::chrono::time_point_cast<std::chrono::milliseconds>(tp_now);
std::cout << tp_now_millisec.time_since_epoch().count() << std::endl;
//1674105484826

auto tp_now_hour = std::chrono::time_point_cast<std::chrono::hours>(tp_now);
std::cout << tp_now_hour.time_since_epoch().count() << std::endl;
//465029
```

# 时钟 Clock

`std::chrono::system_clock`表示当前的系统时钟，系统中运行的所有进程使用该时钟得到的时间是一致的；
除了`system_clock`外，还有`steady_clock`（等同于`high_resolution_clock`），每种clock都有各自的Rep，Period，duration和time_point；

```c++
now() //获取当前时间的time_point
to_time_t(tp) //将time_point转换为time_t类型的秒数
from_time_t(timestamp) //将time_t类型的秒数转为time_point
```

时间戳解析：
```c++
auto timestamp = std::chrono::system_clock::to_time_t(tp_now_sec);
std::cout << timestamp << std::endl;
//1674107006

char datetime[30];
ctime_s(datetime, sizeof(datetime), &timestamp);
std::cout << datetime << std::endl;
//Thu Jan 19 13:43:26 2023

std::tm tm_data;
localtime_s(&tm_data, &timestamp);
printf("%d:%d:%d\n", tm_data.tm_year + 1900, tm_data.tm_mon + 1, tm_data.tm_mday);
//2023:1:19
```

# 常见用法

## 获取当前时间

```cpp
#include <iostream>
#include <chrono>
#include <ctime>

int main() {
    auto now = std::chrono::system_clock::now();
    std::time_t now_c = std::chrono::system_clock::to_time_t(now);
    std::cout << "当前时间: " << std::ctime(&now_c);
    return 0;
}
```

## 测量代码执行时间

```cpp
#include <iostream>
#include <chrono>

int main() {
    auto start = std::chrono::high_resolution_clock::now();
    
    // 放置需要测量执行时间的代码
    for (int i = 0; i < 1000000; ++i) {}

    auto end = std::chrono::high_resolution_clock::now();
    std::chrono::duration<double> elapsed = end - start;
    std::cout << "执行时间: " << elapsed.count() << "秒" << std::endl;
    return 0;
}
```

## 表示特定时间长度

```cpp
#include <iostream>
#include <chrono>

int main() {
    std::chrono::hours halfDay(12);
    std::chrono::minutes halfHour(30);
    std::chrono::seconds fiveMinutes(300);
    
    auto total = halfDay + halfHour + fiveMinutes;
    std::cout << "总时间: " << std::chrono::duration_cast<std::chrono::minutes>(total).count() << "分钟" << std::endl;
    return 0;
}
```