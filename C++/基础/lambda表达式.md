*lambda表达式是定义匿名函数对象的简便方法*
*除此之外还具有捕获上下文变量的能力*

# 语法

分为==捕获列表==，参数列表，可变规格，异常说明，返回类型，==函数体==，其中非高亮的均可以省略；
```text
[capture list](parameters) mutable throw() ->return-type {statement}
```

## 捕获列表

又称为lambda导入期，能够捕获上下文中的变量以供lambda函数使用；

|列表形式|含义|
|:-:|:-|
|[]|不捕获任何变量|
|[var]|以值传递的方式捕获变量`var`|
|[=]|以值传递的方式捕获所有父作用域的变量，包括`this`指针|
|[&var]|以引用传递的方式捕获变量`var`|
|[&]|以引用传递的方式捕获所有父作用域的变量，包括`this`指针|
|[this]|以值传递的方式捕获当前的`this`指针|
|[&,=var,this]|列表形式可以组合，但是不允许变量的重复捕获|

## 参数列表

如果lambda函数不需要参数，可以连带`()`一起省略；

## 可变规格

默认情况下lambda函数总是`const`函数，使用`mutable`修饰符可以取消lambda函数的常量性；
在使用该修饰符时，即使函数参数为空，也不可以省略`()`；

```c++
int nVal = 10;
auto add = [nVal](int x, int y) mutable
{
	nVal++; //虽然值传递了nVal，但因为使用mutable修饰，因此可以修改nVal
	return x + y; 
};
```

## 异常说明

用于lambda函数内部抛出异常；
不建议使用thow处理异常，PASS；

## 返回类型

在返回类型明确（编译器能够自行推导）和不需要返回值的情况下，可以连带`->`一起省略；

## 函数体

除了可以使用参数列表中的参数外，还可以使用捕获的变量；


# 应用场景

```c++
std::vector<int> a = { 10, 9, 8, 7 };

//排序
std::sort(a.begin(), a.end(), [](int x, int y) {return x < y; });

//遍历
std::for_each(a.begin(), a.end(), [](int x) {std::cout << x << std::endl; }); 
// 7 8 9 10
```
