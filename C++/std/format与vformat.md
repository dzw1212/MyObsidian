# format

C++20新引入`std::format`用以格式化字符串，旨在替代之前的C风格的`sprintf`和C++风格的`stringstream`方法；

```cpp
#include <format>

std::string result = std::format("The answer is {}, {.2f}.", 42, 3.1415);
```

具体使用方法参考：[[fmt格式]]

# vformat

`std::vformat` 是 C++20 引入的一部分，用于格式化字符串。它是 `<format>` 头文件中定义的一种功能，允许使用格式化字符串和参数列表来创建格式化后的字符串；

```cpp
#include <format>
#include <iostream>
#include <string>

int main() {
    std::string format_string = "Hello, {}! You have {} new messages.";
    std::string name = "Alice";
    int message_count = 5;

    // 使用 std::vformat
    std::string result = std::vformat(format_string, std::make_format_args(name, message_count));

    std::cout << result << std::endl;

    return 0;
}
```

# 区别

假如我需要一个函数，其中使用`fmt`根据输入的参数构建字符串，这种情况下就需要`vformat`的灵活性；

`std::format`的第一个参数，即格式字符串，必须是编译时常量，需要在编译时就已经确定而不是运行时才确定，即使这个值是`const std::string`或`std::string_view`类型也不行；

```cpp
#include <iostream>
#include <format>

void print(const std::string& msg)
{
	std::cout << std::format(msg);
	//error C7595: “std::_Basic_format_string<char>::_Basic_format_string”: 对即时函数的调用不是常量表达式
}

int main()
{
	print("hello");
	return 0;
}
```

以下使用`vformat`，允许格式字符串通过参数传入：

```cpp
#include <iostream>
#include <format>
#include <string_view>

template<typename ...Args>
void print(const std::string& formatStr, Args&&... args)
{
	auto sss = std::vformat(formatStr, std::make_format_args(std::forward<Args>(args)...));

	std::cout << sss;
}

int main()
{
	print("这是我的{0}{1}。", "000", "00999");
	return 0;
}
```