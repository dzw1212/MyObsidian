*https://fmt.dev/latest/syntax.html#formatexamples

## 参数匹配

```c++
fmt::format("{0}, {1}, {2}", 'a', 'b', 'c');
// Result: "a, b, c"
fmt::format("{}, {}, {}", 'a', 'b', 'c');
// Result: "a, b, c"
fmt::format("{2}, {1}, {0}", 'a', 'b', 'c');
// Result: "c, b, a"
fmt::format("{0}{1}{0}", "abra", "cad");  // arguments' indices can be repeated
// Result: "abracadabra"
```

## 对齐

```c++
//静态设置对齐参数
fmt::format("{:<30}", "left aligned");
// Result: "left aligned                  "
fmt::format("{:>30}", "right aligned");
// Result: "                 right aligned"
fmt::format("{:^30}", "centered");
// Result: "           centered           "
fmt::format("{:*^30}", "centered");  // use '*' as a fill char
// Result: "***********centered***********"

//动态设置对齐参数
fmt::format("{:<{}}", "left aligned", 30);
// Result: "left aligned
```

## 正负号

```c++
fmt::format("{:+f}; {:+f}", 3.14, -3.14);  // show it always
// Result: "+3.140000; -3.140000"
fmt::format("{: f}; {: f}", 3.14, -3.14);  // show a space for positive numbers
// Result: " 3.140000; -3.140000"
fmt::format("{:-f}; {:-f}", 3.14, -3.14);  // show only the minus -- same as '{:f}; {:f}'
// Result: "3.140000; -3.140000"
```

## 格式转换

```c++
fmt::format("int: {0:d};  hex: {0:x};  oct: {0:o}; bin: {0:b}", 42);
// Result: "int: 42;  hex: 2a;  oct: 52; bin: 101010"
// with 0x or 0 or 0b as prefix:
fmt::format("int: {0:d};  hex: {0:#x};  oct: {0:#o};  bin: {0:#b}", 42);
// Result: "int: 42;  hex: 0x2a;  oct: 052;  bin: 0b101010"
```

## 矩形框

```c++
fmt::print(
  "┌{0:─^{2}}┐\n"
  "│{1: ^{2}}│\n"
  "└{0:─^{2}}┘\n", "", "Hello, world!", 20);

┌────────────────────┐
│   Hello, world!    │
└────────────────────┘
```

## 日期时间

```c++
#include <fmt/chrono.h>

auto t = tm();
t.tm_year = 2010 - 1900;
t.tm_mon = 7;
t.tm_mday = 4;
t.tm_hour = 12;
t.tm_min = 15;
t.tm_sec = 58;
fmt::print("{:%Y-%m-%d %H:%M:%S}", t);
// Prints: 2010-08-04 12:15:58
```

## 千位分隔符

```c++
#include <fmt/format.h>

auto s = fmt::format(std::locale("en_US.UTF-8"), "{:L}", 1234567890);
// s == "1,234,567,890"
```

