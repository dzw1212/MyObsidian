*https://github.com/HappyCerberus/book-cpp-algorithms?tab=License-1-ov-file*

# 标准库算法的演化

C++98中，很多算法就已经提出了，但是不好用；
以`std::for_each`为例子，
```cpp
std::for_each(begin_iter, end_iter, func)
```

C++11后，算法可以配合`lambda`函数使用，增加了实用性；
```cpp
std::for_each(begin_iter, end_iter, [](){})
```

C++17引入了执行策略[[std-execution]]等补丁，以便在多核处理器上更高效；
```cpp
std::for_each(std::execution::par_unseq, begin_iter, end_iter, func)
```

c++20引入了`ranges`与`views`两种不同版本的算法，`ranges`提供了一种更现代和一致的方式来处理范围，而`views`提供了一种惰性计算（只有在需要的时候才计算）的方式来操作和转换范围。它允许你在不实际修改底层数据的情况下，创建数据的视图；
```cpp
std::vector<int> data = {1, 2, 3, 4, 5};

// 使用 std::ranges::for_each
std::ranges::for_each(data, [](int n) {
	std::cout << n << " ";
});
```

```cpp
std::vector<int> data = {1, 2, 3, 4, 5};

// 组合多个视图操作，形成一个管道式的操作链
// 过滤出偶数并将其平方
auto even_squares = data | std::views::filter([](int n) { return n % 2 == 0; })
						 | std::views::transform([](int n) { return n * n; });

for (int n : even_squares) {
	std::cout << n << " ";  // 输出: 4 16
}
```