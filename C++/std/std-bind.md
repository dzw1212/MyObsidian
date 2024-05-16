`std::bind`用来绑定函数调用的某些参数；
类似于函数的默认参数，但不像默认参数那样需要放到所有参数的最后；
将可调用对象保存起来，在需要的时候调用，以实现**延迟计算**；

std::bind一般配合占位符`std::placeholders`使用；
```c++
#include <functional>

int TestBindFunc(int A, std::string B, float C)
{
	std::cout << A << std::endl;
	std::cout << B << std::endl;
	std::cout << C << std::endl;

	return A;
}

int main()
{
	auto bindFunc1 = std::bind(TestBindFunc, std::placeholders::_1, "BBB", .5f);
	auto bindFunc2 = std::bind(TestBindFunc, std::placeholders::_1,
		std::placeholders::_2, std::placeholders::_3);

	bindFunc1(10);
	bindFunc2(10, "bbb", .4f);

	return 0;
}
```