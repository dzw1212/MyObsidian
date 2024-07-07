`std::iterator_traits` 是 C++ 标准库中的一个模板结构体，它提供了一种方式来访问迭代器的属性，比如迭代器指向的值的类型、迭代器的类别等。这对于编写泛型代码非常有用，因为你可以编写依赖于迭代器属性的代码，而不必关心迭代器的具体类型。



`std::iterator_traits` 定义在头文件 `<iterator>` 中，它为所有迭代器定义了五种类型别名：

- `difference_type`：迭代器之间的差异类型；
- `value_type`：迭代器指向的值的类型；
- `pointer`：指向迭代器指向的值的指针类型；
- `reference`：迭代器指向的值的引用类型；
- `iterator_category`：迭代器的类别，如输入迭代器、输出迭代器、前向迭代器、双向迭代器、随机访问迭代器；

---

```cpp
template<typename IterT>
struct iterator_traits;
```

它的运作方式是（以类型标签为例）：针对每一个类型`IterT`，在其内一定要通过`typedef`或`using`声明一个类型，名为`iterator_category`，用来作为迭代器类型标签；其他类型别名也是如此；

对于一个`deque`迭代器来说其结构可能如此：

```cpp
template<...>
class deque
{
public:
	class iterator
	{
	public:
		typedef random_access_iterator_tag iterator_category;
	};
};
```
# 获取类型信息

假设你正在编写一个泛型函数，该函数需要根据迭代器指向的对象类型来处理数据。你可以使用 `std::iterator_traits` 来获取迭代器指向的对象类型：

```cpp
#include <iostream>
#include <vector>
#include <iterator>

template<typename Iter>
void print_elements(Iter begin, Iter end) {
    using ValueType = typename std::iterator_traits<Iter>::value_type;
    //或者typedef typename std::iterator_traits<Iter>::value_type ValueType
    
    for (Iter it = begin; it != end; ++it) {
        const ValueType& value = *it;
        std::cout << value << std::endl;
    }
}

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};
    print_elements(vec.begin(), vec.end()); //此处ValueType即为int
    return 0;
}
```

# 设置类型信息

以下是一个简单的示例，展示了如何为一个自定义的`Container`类实现一个支持`iterator_traits`的迭代器：

```cpp
#include <iterator>

class Container {
public:
    class Iterator {
    public:
        using difference_type = std::ptrdiff_t;
        using value_type = int; // 假设容器存储int类型
        using pointer = value_type*;
        using reference = value_type&;
        using iterator_category = std::forward_iterator_tag;

        Iterator(pointer ptr) : m_ptr(ptr) {}

        reference operator*() const { return *m_ptr; }
        pointer operator->() { return m_ptr; }

        // 前缀++
        Iterator& operator++() {
            m_ptr++;
            return *this;
        }

        // 后缀++
        Iterator operator++(int) {
            Iterator tmp = *this;
            ++(*this);
            return tmp;
        }

        friend bool operator==(const Iterator& a, const Iterator& b) { return a.m_ptr == b.m_ptr; }
        friend bool operator!=(const Iterator& a, const Iterator& b) { return a.m_ptr != b.m_ptr; }

    private:
        pointer m_ptr;
    };

    // Container的成员函数，包括begin()和end()等
    Iterator begin() { /* 实现 */ }
    Iterator end() { /* 实现 */ }
};
```

```cpp
using IterCatrgory = typename std::iterator_traits<Container::Iterator>::iterator_category;
bool bIsForward = std::is_base_of<IterCatrgory, std::forward_iterator_tag>::value;

if (bIsForward)
	...
```

## is_base_of与typeid

两者都可以查询类型信息，但是两者用于不同的场景：

- `typeid`是一个运算符，用于在**运行时获取**一个表达式的类型信息；
- 它返回一个`std::type_info`对象的引用，该对象代表了表达式的类型；
- `typeid`可以用于获取类型的名称（通过`name()`成员函数）或比较两个类型是否相同（通过`operator==`）；
- `typeid`主要用于运行时类型识别（`RTTI`）；


- `std::is_base_of`是一个模板结构体，用于**编译时检查**一个类型是否是另一个类型的基类。
- 它是类型特征（`type traits`）的一部分，提供了一个静态成员`value`，该成员是一个布尔值，表示第一个模板参数类型是否是第二个模板参数类型的基类或与之相同；
- `std::is_base_of`主要用于编译时的类型检查和元编程；

