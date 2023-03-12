chrono是一个时间库，
头文件为`#include <chrono>`，一般还需要`#include <ratio>`；

# 时间段 duration

```c++
template <class Rep, class Period = ratio<1> > class duration;

//Rep为数值类型，如int,float,double
//Period表示“用秒定义的时间单位”

template <intmax_t N, intmax_t D = 1> class ratio;

//std::ratio表示一个分数值，N为分子(numerator)，D为分母(denominator)
```

|已定义变量|实际含义|
|:-|:-|
|std::chrono::hours|`std::chrono::duration<int, 3600>|
|minutes|`duration<int, std::ratio<60>>|
|seconds|`duration<long long>|
|milliseconds|`duration<long long, std::milli>|
|nanoseconds|`duration<long long, std::nano>|

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
