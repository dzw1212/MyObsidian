# std::sort

- **用法**: `std::sort`是一个快速排序算法，通常基于快速排序、堆排序或插入排序的混合实现。
- **稳定性**: `std::sort`是不稳定的排序算法，这意味着相同大小的元素在排序后可能会改变它们的相对位置。
- **复杂度**: 平均时间复杂度为 $O(n \log n)$。


```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {4, 2, 5, 3, 1};
    std::sort(vec.begin(), vec.end());

    for (int num : vec) {
        std::cout << num << " ";
    }
    return 0;
}
```

# std::stable_sort

- **用法**: `std::stable_sort`是一个稳定的排序算法，通常基于归并排序。
- **稳定性**: `std::stable_sort`是稳定的，这意味着相同大小的元素在排序后保持它们的相对位置。
- **复杂度**: 平均时间复杂度为 \(O(n \log n)\)，但通常比`std::sort`稍慢。


```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {4, 2, 5, 3, 1};
    std::stable_sort(vec.begin(), vec.end());

    for (int num : vec) {
        std::cout << num << " ";
    }
    return 0;
}
```

# 排序方法

## less和greater

缺省默认使用`std::less`进行升序排列；

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {4, 2, 5, 3, 1};
    std::sort(vec.begin(), vec.end(), std::greater<int>()); // 降序

    for (int num : vec) {
        std::cout << num << " ";
    }
    return 0;
}
```

### 自定义结构体实现

`std::less`与`std::greater`是基于`operator <`和`operator >`实现的，只要自定义的结构体实现了`<`和`>`操作符，就可以通过`less/greater`进行排序；

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <string>

struct Person {
    std::string name;
    int age;

    // 定义 operator< 用于比较
    bool operator<(const Person& other) const {
        return age < other.age; // 按年龄升序排序
    }
};

int main() {
    std::vector<Person> people = {{"Alice", 30}, {"Bob", 25}, {"Charlie", 35}};

    // 使用 std::greater 进行降序排序
    std::sort(people.begin(), people.end(), std::greater<Person>());

    for (const auto& person : people) {
        std::cout << person.name << ": " << person.age << std::endl;
    }
    return 0;
}
```

## 自定义排序函数

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

// 自定义比较函数
bool customCompare(int a, int b) {
    // 例如：按绝对值升序排序
    return std::abs(a) < std::abs(b);
}

int main() {
    std::vector<int> vec = {-4, 2, -5, 3, 1};
    std::sort(vec.begin(), vec.end(), customCompare);

    for (int num : vec) {
        std::cout << num << " ";
    }
    return 0;
}
```

或者用`lambda`表达式：

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {-4, 2, -5, 3, 1};
    std::sort(vec.begin(), vec.end(), [](int a, int b) {
        return std::abs(a) < std::abs(b); // 自定义排序规则
    });

    for (int num : vec) {
        std::cout << num << " ";
    }
    return 0;
}
```

