*用来解决函数返回值的歧义问题*

# Quick Start

```c++
struct QuadVertex
{
	QuadVertex(const std::string& strName)
		: name(strName) {}
	std::string name;
};

std::optional<QuadVertex> FindQuarVertex()
{
	if (true)
		return std::make_optional<QuadVertex>("hello c++17");
	else
		return std::nullopt;
}


int main(int argc, char* argv[])
{
	auto res = FindQuarVertex();
	if (!res)
		std::cout << "error" << std::endl;
	auto val = res.value(); //or *res
	std::cout << val.name << std::endl; //hello c++17

	system("pause");
	return 0;
}
```

# 初始化

```c++
//初始化为空值
std::optional<int> emptyInt;
std::optional<float> emptyFloat = std::nullopt;

//使用实值初始化
std::optional<int> intOpt{10};
std::optional floatOpt{10.f}; //自动推断类型

//使用make_optional进行构造
auto doubleOpt = std::make_optional(10.0);

//使用std::in_place
std::optional<std::vector<int>> vecOpt{std::in_place, {1, 2, 3}};
```

## 为什么使用std::in_place

```c++
std::optional<Example> exampleOpt; //不会调用构造函数

std::optional<>

std::optional<Example> exampleOpt2{std::in_place}; //调用构造函数

auto exampleOpt3 = std::make_optional<Example>(); //调用构造函数

std::optional<Example> exampleOpt4{ Example() }; //额外多了移动拷贝和析构开销
```

# 比较大小

即使被封装进了std::optional，也可以直接使用`<`，`>`，`=`来比较大小，对于空值（std::nullopt），比较大小时永远是小的一方；
