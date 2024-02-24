# 迭代器类型

根据其功能和支持的操作可以分为五种主要类型：

1. 输入迭代器（`Input Iterators`）：只能向前移动（单步递增），支持读取指向的元素，但不支持写入。主要用于从容器中读取数据，如`istream_iterators`；

2. 输出迭代器（`Output Iterators`）：也只能向前移动（单步递增），支持写入指向的元素，但不支持读取。主要用于向容器中写入数据，如`ostream_iterators`；

3. 前向迭代器（`Forward Iterators`）：支持向前移动，可以读写指向的元素，并且可以多次遍历容器中的元素，如`单向linked_list`；

4. 双向迭代器（`Bidirectional Iterators`）：支持向前和向后移动（即可以递增也可以递减），可以读写指向的元素，适用于那些可以双向遍历的容器，如`set/multiset`，`map/multimap`；

5. 随机访问迭代器（`Random Access Iterators`）：提供最丰富的功能，支持所有标准迭代器操作，包括单步递增或递减、元素读写、跳跃访问（通过加减操作符直接访问任意距离的元素），以及比较迭代器之间的距离。适用于像数组这样可以随机访问的容器，如`vector`，`string`，`deque`；

每种迭代器类型都支持一组特定的操作集，这些操作定义了迭代器和容器之间的接口。随着迭代器类型从输入迭代器到随机访问迭代器的递进，支持的操作也越来越多，功能也越来越强大。

# 迭代器类型标签

用于标识迭代器的类型；这些标签是类型系统的一部分，允许函数模板在编译时根据迭代器的类型进行特化，从而选择最合适的实现。这种机制是通过模板特化和函数重载实现的，使得算法可以根据迭代器的能力选择不同的执行路径，优化性能或提供特定的功能；

定义在头文件`<iterator>`中，主要有以下几种：

1. `std::input_iterator_tag`：表示迭代器是一个输入迭代器，支持读取操作，可以逐个向前移动。

2. `std::output_iterator_tag`：表示迭代器是一个输出迭代器，支持写入操作，也可以逐个向前移动，但不支持读取。

3. `std::forward_iterator_tag`：继承自`std::input_iterator_tag`，表示迭代器是一个前向迭代器，支持读写操作，可以多次遍历容器中的元素，每次只向前移动一步。

4. `std::bidirectional_iterator_tag`：继承自`std::forward_iterator_tag`，表示迭代器是一个双向迭代器，除了支持前向迭代器的所有操作外，还可以向后移动。

5. `std::random_access_iterator_tag`：继承自`std::bidirectional_iterator_tag`，表示迭代器是一个随机访问迭代器，支持直接访问任何元素，支持迭代器的加减操作、元素的随机访问、以及迭代器之间的距离计算。

类型标签可以通过`iterator_traits`中的`iterator_category`进行获取；

[[iterator_traits]]

