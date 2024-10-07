# std::multimap

`multimap`相比`map`，一个键可以对应多个值，因此不能直接用`[]`来取对应的值，一般使用`equal_range/lower_bound/upper_bound`来取对应的值；

```cpp
#include <iostream>
#include <map>
#include <string>

int main() {
    // 创建一个 multimap，其中键是 string 类型，值也是 string 类型
    std::multimap<std::string, std::string> phoneBook;

    // 向 multimap 中添加元素
    phoneBook.insert(std::make_pair("Alice", "123-456-7890"));
    phoneBook.insert(std::make_pair("Alice", "234-567-8901"));
    phoneBook.insert(std::make_pair("Bob", "345-678-9012"));
    phoneBook.insert(std::make_pair("Alice", "456-789-0123"));

    // 打印所有 Alice 的电话号码
    std::cout << "Alice's phone numbers:" << std::endl;
    auto range = phoneBook.equal_range("Alice");
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << it->second << std::endl;
    }

    // 使用 lower_bound 和 upper_bound
    std::cout << "Alice's phone numbers:" << std::endl;
    auto itLow = phoneBook.lower_bound("Alice");  // 第一个 "Alice"
    auto itHigh = phoneBook.upper_bound("Alice"); // 第一个不是 "Alice" 的元素

    for (auto it = itLow; it != itHigh; ++it) {
        std::cout << it->second << std::endl;
    }

    // 遍历并打印所有电话号码
    std::cout << "\nAll phone numbers:" << std::endl;
    for (const auto& entry : phoneBook) {
        std::cout << entry.first << ": " << entry.second << std::endl;
    }

    return 0;
}
```

# std::multiset

`multiset`相比`set`，允许存在多个重复的元素，但是整体还是有序的；

## 应用1: 统计元素频率

```cpp
#include <iostream>
#include <set>

int main() {
    std::multiset<int> numbers = {1, 2, 2, 3, 3, 3, 4, 4, 4, 4};

    // 统计每个数字的出现次数
    for (int num = 1; num <= 4; ++num) {
        std::cout << "Number " << num << " appears " << numbers.count(num) << " times." << std::endl;
    }

    return 0;
}
```

## 应用2: 处理并集、交集

```cpp
#include <iostream>
#include <set>
#include <algorithm>
#include <iterator>

int main() {
    // 定义两个 multiset
    std::multiset<int> set1 = {1, 2, 2, 3, 4};
    std::multiset<int> set2 = {2, 3, 3, 4, 5, 6};

    // 容器用于存储结果
    std::multiset<int> unionResult;
    std::multiset<int> intersectionResult;

    // 计算并集
    std::set_union(set1.begin(), set1.end(),
                   set2.begin(), set2.end(),
                   std::inserter(unionResult, unionResult.begin()));

    // 计算交集
    std::set_intersection(set1.begin(), set1.end(),
                          set2.begin(), set2.end(),
                          std::inserter(intersectionResult, intersectionResult.begin()));

    // 输出并集结果
    std::cout << "Union of set1 and set2: ";
    for (int num : unionResult) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    // 输出交集结果
    std::cout << "Intersection of set1 and set2: ";
    for (int num : intersectionResult) {
        std::cout << num << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

---

除此之外，如果不需要自动排序，可以使用`unordered_multimap`和`unordered_multiset`；

