`std::any`是C++17标准库的一部分，**提供了一种类型安全的方式来存储和操作任意类型的值**，它可以存储单个值，这个值可以是任何类型，**只要该类型满足复制构造和析构的条件**；`std::any`主要用于需要处理不确定类型数据的场景；

1. 存储值：你可以通过构造函数或`std::any`的`emplace`方法来存储值；

2. 访问值：要访问`std::any`中存储的值，你可以使用`std::any_cast`。如果尝试将`std::any`对象转换为与其存储值不兼容的类型，这个操作可能抛出`std::bad_any_cast`异常；

3. 检查是否有值：使用`has_value`方法可以检查`std::any`对象是否存储了值；

4. 移除值：使用`reset`方法可以移除存储的值，使`std::any`对象变为空；

```cpp
#include <any>
#include <iostream>
#include <string>

void ProcessAny(const std::any& value) //通过const ref传参
{
	...
}

int main() {
	std::any a = 10; // 存储int
    std::cout << std::any_cast<int>(a) << std::endl; // 访问值

    a = std::string("Hello, std::any!"); // 存储string
    std::cout << std::any_cast<std::string>(a) << std::endl; // 访问值

    // 尝试安全访问值
    try {
        std::cout << std::any_cast<double>(a) << std::endl; // 错误的类型转换
    } catch (const std::bad_any_cast& e) {
        std::cout << e.what() << std::endl; // 捕获并处理异常
    }

    if (a.has_value()) {
        std::cout << "a has a value." << std::endl;
    }

    a.reset(); // 移除值
    if (!a.has_value()) {
        std::cout << "a is now empty." << std::endl;
    }

    return 0;
}
```

---

默认情况下，如果转换的类型不匹配，会抛出一个 `std::bad_any_cast` 异常。如果你想避免使用异常机制来处理类型不匹配的情况，可以使用 `std::any::type()` 方法来检查 `std::any` 对象当前存储的类型是否与预期匹配，或者使用 `std::any_cast` 的指针版本来尝试转换而不抛出异常；

```cpp
// 检查a是否包含int类型的值
if (a.type() == typeid(int)) {
	std::cout << "a contains an int" << std::endl;
	// 安全地使用std::any_cast
	int value = std::any_cast<int>(a);
	std::cout << "Value: " << value << std::endl;
} else {
	std::cout << "a does not contain an int" << std::endl;
}
```

```cpp
// 尝试转换为int指针
int* intPtr = std::any_cast<int>(&a);
if (intPtr != nullptr) {
	std::cout << "a contains an int: " << *intPtr << std::endl;
} else {
	std::cout << "a does not contain an int" << std::endl;
}

// 尝试转换为std::string指针
std::string* strPtr = std::any_cast<std::string>(&a);
if (strPtr != nullptr) {
	std::cout << "a contains a string: " << *strPtr << std::endl;
} else {
	std::cout << "a does not contain a string" << std::endl;
}
```

---

与之类似的有`std::variant`，但是`varient`需要事先写出允许的类型有哪些；[[variant]]
