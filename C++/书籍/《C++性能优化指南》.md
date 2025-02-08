# 字符串优化

## 避免内存再分配

`std::string`的内存管理策略采用一种称为**指数增长**的策略来应对字符串长度的增加。这种策略的具体实现可能因标准库的实现而异，但通常会在需要扩展容量时分配比当前容量大一倍或更多的内存，以减少频繁的内存分配操作。

```cpp
#include <iostream>
#include <string>

int main() {
    std::string str;
    std::cout << "Initial capacity: " << str.capacity() << std::endl;

    for (int i = 0; i < 1000; ++i) {
        str += 'a';
        std::cout << "Length: " << str.length() << ", Capacity: " << str.capacity() << std::endl;
    }

    return 0;
}
```

```txt
Initial capacity: 15
Length: 1, Capacity: 15
...
Length: 15, Capacity: 15
Length: 16, Capacity: 31
...
Length: 31, Capacity: 31
Length: 32, Capacity: 47
...
Length: 47, Capacity: 47
Length: 48, Capacity: 70
...
Length: 70, Capacity: 70
Length: 71, Capacity: 105
...
Length: 105, Capacity: 105
Length: 106, Capacity: 157
...
Length: 157, Capacity: 157
Length: 158, Capacity: 235
...
Length: 235, Capacity: 235
Length: 236, Capacity: 352
...
Length: 352, Capacity: 352
Length: 353, Capacity: 528
...
Length: 528, Capacity: 528
Length: 529, Capacity: 792
...
Length: 792, Capacity: 792
Length: 793, Capacity: 1188
...
Length: 1000, Capacity: 1188
```

观察上面的结果，可以看到`std::string`初始分配了**15**的容量，当初次溢出时翻倍，后续再溢出时翻**1.5**倍；

==提前预估字符串的长度并预留存储空间，可以避免内存再分配的开销；==

```cpp
#include <iostream>
#include <string>

#include "Stopwatch.h"

int main() {
    std::string str;
    std::cout << "Initial capacity: " << str.capacity() << std::endl;

    {
        Stopwatch watcher;
        for (int i = 0; i < 1000; ++i) {
            str += 'a';
        }
    } //执行时间: 0.0243ms

    str.clear();

    {
        Stopwatch watcher;
        str.reserve(1000);
        for (int i = 0; i < 1000; ++i) {
            str += 'a';
        }
    } //执行时间: 0.0157ms

    system("pause");
    return 0;
}
```

## 使用字符串数组

即极端性能限制的情况下，可以使用C风格的字符串数组代替`std::string`，同时也牺牲了灵活性与安全性；

```cpp
#include <iostream>
#include <string>

#include "Stopwatch.h"

int main() {
    std::string str;
    std::cout << "Initial capacity: " << str.capacity() << std::endl;

    {
        Stopwatch watcher;
        for (int i = 0; i < 1000; ++i) {
            str += 'a';
        } 
    } //执行时间: 0.0303ms

    char szStr[1000];
    memset(szStr, 0, 1000);

    {
        Stopwatch watcher;
        str.reserve(1000);
        for (int i = 0; i < 1000; ++i) {
            szStr[i] = 'a';
        } 
    } //执行时间: 0.0007ms

    system("pause");
    return 0;
}
```



# 循环写法对比

```cpp
#include <iostream>
#include <string>
#include <vector>

#include "Stopwatch.h"

int main() {
    std::vector<int> vec;
    for (int i = 0; i < 10000; ++i)
        vec.push_back(i);

    {
        Stopwatch watcher;
        for (size_t i = 0; i < vec.size(); ++i) {}
    } //执行时间: 0.0138ms

    {
        Stopwatch watcher;
        for (auto it = vec.begin(); it != vec.end(); ++it) {}
    } //执行时间: 0.7132ms

    {
        Stopwatch watcher;
        for (auto it = vec.cbegin(); it != vec.cend(); ++it) {}
    } //执行时间: 0.784ms

    {
        Stopwatch watcher;
        for (auto it : vec) {}
    } //执行时间: 0.0049ms

    {
        Stopwatch watcher;
        for (const auto& it : vec) {}
    } //执行时间: 0.0041ms

    {
        Stopwatch watcher;
        for (auto it = vec.begin(), end = vec.end(); it != end; ++it) {}
    } //执行时间: 0.2262ms

    system("pause");

    return 0;
}
```

如果需要修改数据，最佳写法是`for (auto it : vec)`；如果不需要修改数据，最佳写法是`for (const auto& it : vec)`；

# shared_ptr

## 使用make_shared代替new

1. `std::make_shared` 在单次内存分配中同时分配对象和控制块（引用计数等），而使用 `new` 创建 `std::shared_ptr` 需要两次内存分配：一次分配对象，另一次分配控制块。减少内存分配次数可以提高性能。

2. 使用 `new` 创建对象时，如果在 `std::shared_ptr` 构造函数之前抛出异常，可能会导致内存泄漏。`std::make_shared` 可以确保在创建对象和控制块时不会发生内存泄漏。

```cpp
std::shared_ptr<int> ptr(new int(10)); //不推荐

auto ptr = std::make_shared<int>(10); //推荐
```

## 或许没必要传参shared_ptr

当`shared_ptr`作为参数传入时，会增加引用计数，函数结束后会减少引用计数，增加和减少`shared_ptr`中的引用计数并不是执行一个简单的增量指令，而是使用完整的内存屏障进行一次非常昂贵的原子性增加操作，这样`shared_ptr`才能工作于多线程程序中；

传递`shared_ptr`的好处是，不需要由外部确保指针的可用性，但是大多数情况下，指针的可用性不会在函数调用期间发生变化，此时传原始指针或引用是更好的选择；

- 传原始指针：可以传空值，可以改变指向的对象，但需要手动管理内存；

- 传引用：语法简洁，性能高，但无法传递空值，无法改变引用的对象；

```cpp
#include <iostream>
#include <memory>

class MyClass {
public:
    void display() const {
        std::cout << "MyClass instance" << std::endl;
    }
};

void byReference(MyClass& obj) {
    obj.display();
}

void byPointer(MyClass* obj) {
    if (obj) {
        obj.display();
    }
}

void bySharedPtr(std::shared_ptr<MyClass> obj) {
    if (obj) {
        obj->display();
    }
}

int main() 
{
    auto sharedPtr = std::make_shared<MyClass>();
    bySharedPtr(sharedPtr);

	byPoint(sharedPtr->get());

	byReference(*sharedPtr->get());

    return 0;
}
```

# 避免不必要的复制行为

复制行为可能存在于：
1. 类的拷贝构造函数和赋值构造函数；
2. 函数传参时；
3. 函数返回时；
4. 插入元素到STL容器中（如果是`vector`且超出大小令`vector`重新分配内存，则所以成员都会通过移动构造函数或复制构造函数复制到新申请的`vector`中，开销非常恐怖）；

## 禁用复制构造

对于类内部某些不希望被复制的成员，可以直接禁用复制；

```cpp
class MyClass
{
public:
    MyClass() = default;
    MyClass(MyClass&) = delete;
    MyClass& operator=(const MyClass&) = delete;
};

MyClass a;
MyClass b(a); //Error
MyClass c = a; //Error
```

## 使用const&传参

 ## 返回引用

函数返回引用可以避免复制构造函数，但需要注意的是，不能返回局部变量的引用，可以返回引用的有：
1. 类成员的引用；
2. 静态变量的引用；
3. 传入参数的引用；

## 使用传出参数而不是返回值

```cpp
std::vector<Point> GetAllPoints(int nIndex); //不推荐

bool GetAllPoints(int nIndex, std::vector<Point>& vecPoints); //推荐
```

# 优化查找

`std::find`属于线性查找，不需要数据已排序，时间复杂度是`O(n)`；

`std::find_if`与`std::find`类似，但是允许自定义比较条件；

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

// 使用 std::find_if 查找第一个大于 3 的元素
auto it = std::find_if(vec.begin(), vec.end(), [](int value) {
	return value > 3;
});
```

`std::binary_search`不返回值，仅判断该值是否存在，其复杂度为`O(log n)`，但是前提是数据已排序；

```cpp
std::vector<int> vec = {1, 2, 3, 4, 5};

// 使用 std::binary_search
bool found = std::binary_search(vec.begin(), vec.end(), 3);
```

`std::equal_range`用于在有序容器中查找等于特定值的元素范围，它返回一对迭代器，分别指向第一个不小于给定值的元素和第一个大于给定值的元素，复杂度为`O(log n)`；

```cpp
std::vector<int> vec = { 1, 2, 2, 2, 3, 4, 5 };

// 使用 std::equal_range 查找等于 2 的元素范围
auto range = std::equal_range(vec.begin(), vec.end(), 2);

bool bFind = range.first != range.second;
```

`std::lower_bound`返回一个迭代器，指向第一个不小于（即大于或等于）给定值的元素；
`std::upper_bound`返回一个迭代器，指向第一个大于给定值的元素；

```cpp
// 使用 std::lower_bound 查找键为 5 的元素
auto it = myMap.lower_bound(5);

if (it != myMap.end() && it->first == 5) {
```

## 各查找方法性能对比

- 对于有序容器：

```cpp
int main() 
{
    std::map<int, std::string> myMap = {
        {1, "one"},
        {2, "two"},
        {3, "three"},
        {4, "four"},
        {6, "six"},
        {9, "nine" },
        {11, "eleven" },
        {100, "hundred" },
    };

    {
        Stopwatch watcher;

        bool bFind = myMap.contains(5);
    } //执行时间: 0.0005ms

    {
        Stopwatch watcher;

        bool bFind = myMap.find(5) != myMap.end();
    } //执行时间: 0.0014ms

    {
        Stopwatch watcher;

        bool bFind = std::find(myMap.begin(), myMap.end(), std::make_pair(5, "five")) != myMap.end();
    } //执行时间: 0.0019ms

    {
        Stopwatch watcher;

        bool bFind = std::binary_search(myMap.begin(), myMap.end(), std::make_pair(5, "five"));
    } //执行时间: 0.0026ms

    {
        Stopwatch watcher;

        auto range = std::equal_range(myMap.begin(), myMap.end(), std::make_pair(5, "five"));
        bool bFind = range.first != range.second;
    } //执行时间: 0.0023ms

    {
        Stopwatch watcher;

        auto it = std::lower_bound(myMap.begin(), myMap.end(), std::make_pair(5, "five"));
        bool bFind = it != myMap.end() && (it->first == 5);
    } //执行时间: 0.0024ms

    return 0;
}
```

- 对于无序容器：

```cpp
int main() 
{
    {
        Stopwatch watcher;
        std::vector<int> myMap = { 10, 120, 15, 1515, 99, 1, 0, 0, 5, 6, 8 };
        bool bFind = std::find(myMap.begin(), myMap.end(), 5) != myMap.end();
    } //执行时间: 0.0896ms

    {
        Stopwatch watcher;
        std::vector<int> myMap = { 10, 120, 15, 1515, 99, 1, 0, 0, 5, 6, 8 };
        std::sort(myMap.begin(), myMap.end());
        bool bFind = std::binary_search(myMap.begin(), myMap.end(), 5);
    } //执行时间: 0.023ms

    {
        Stopwatch watcher;
        std::vector<int> myMap = { 10, 120, 15, 1515, 99, 1, 0, 0, 5, 6, 8 };
        std::sort(myMap.begin(), myMap.end());
        auto range = std::equal_range(myMap.begin(), myMap.end(), 5);
        bool bFind = range.first != range.second;
    } //执行时间: 0.0133ms

    {
        Stopwatch watcher;
        std::vector<int> myMap = { 10, 120, 15, 1515, 99, 1, 0, 0, 5, 6, 8 };
        std::sort(myMap.begin(), myMap.end());
        auto it = std::lower_bound(myMap.begin(), myMap.end(), 5);
        bool bFind = it != myMap.end() && (*it == 5);
    } //执行时间: 0.0084ms

    return 0;
}
```

## 查找方法总结

- 对于有序容器，直接使用容器自带的`contains`方法，如果需要获取值则`find`方法；
- 对于无序容器，如果排序成本不高，推荐排序后再使用`lower_bound`；

# vector优化

## 避免内存再分配

类似字符串一样（其实`c11`之后字符串就是通过`vector`实现的），提前`reserve`容器的空间，能够避免内存再分配+移动元素+释放原空间的性能损耗；

```cpp
{
	Stopwatch watcher;
	std::vector<int> myVec;
	for (int i = 0; i < 10000; ++i)
	{
		myVec.push_back(i);
	} //执行时间: 1.62ms
}

{
	Stopwatch watcher;
	std::vector<int> myVec;
	myVec.reserve(10000);
	for (int i = 0; i < 10000; ++i)
	{
		myVec.push_back(i);
	} //执行时间: 1.2343ms
}
```

## 避免中间插入元素

在前端插入元素是低效的，因为需要复制 `vector` 中的所有元素来为新元素腾出空间；
如果需要在前端插入和删除，请使用`std::deque`或`std::list`；

```cpp	
{
	Stopwatch watcher;
	for (int i = 0; i < 100; ++i)
		myVec.push_back(100);
} //执行时间: 0.0154ms

{
	Stopwatch watcher;
	for (int i = 0; i < 100; ++i)
		myVec.insert(myVec.end(), 100);
} //执行时间: 0.1459ms

{
	Stopwatch watcher;
	for (int i = 0; i < 100; ++i)
		myVec.insert(myVec.begin(), 100);
} //执行时间: 0.7467ms

```


