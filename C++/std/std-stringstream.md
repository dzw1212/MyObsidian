`std::stringstream` 是 C++ 标准库中的一个类，用于在内存中读写字符串流。它继承自 `std::iostream`，因此可以使用标准的输入输出操作。以下是 `std::stringstream` 的一些常见用法：

# 基本用法

```cpp
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::stringstream ss;
    ss << "Hello, " << "world!" << 123;

    std::string result = ss.str();
    std::cout << result << std::endl; // 输出: Hello, world!123

    return 0;
}
```

# 从字符串中读取数据

```cpp
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::string input = "123 456 789";
    std::stringstream ss(input);

    int a, b, c;
    ss >> a >> b >> c;

    std::cout << a << " " << b << " " << c << std::endl; // 输出: 123 456 789

    return 0;
}
```

# 清空和重用

```cpp
#include <iostream>
#include <sstream>
#include <string>

int main() {
    std::stringstream ss;
    ss << "First string";
    std::cout << ss.str() << std::endl; // 输出: First string

    ss.str(""); // 清空内容
    ss.clear(); // 重置状态标志

    ss << "Second string";
    std::cout << ss.str() << std::endl; // 输出: Second string

    return 0;
}
```

# 格式化输出

```cpp
#include <iostream>
#include <sstream>
#include <iomanip>

int main() {
    std::stringstream ss;
    ss << std::fixed << std::setprecision(2) << 123.456;

    std::string result = ss.str();
    std::cout << result << std::endl; // 输出: 123.46

    return 0;
}
```
