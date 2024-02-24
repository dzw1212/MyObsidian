`std::function`是一个**函数包装模板**，可以包装函数、函数指针、类成员函数指针或任意类型的函数对象；
std::function对象可以被拷贝和转移，并且可以使用指定的调用特征来直接调用目标元素；
当std::function对象未包裹任何实际可调用的元素时，强行调用会抛出`std::bad_function_call`异常；

## 包装普通函数
```c++
#include <functional>

unsigned int GetAbsMinus(int nVal1, int nVal2)
{
	return abs(nVal1 - nVal2);
}

int main()
{
	std::function<unsigned int(int, int)> func = GetAbsMinus;
	std::cout << func(15, 20) std::endl; //5
	return 0;
}
```

## 包装模板函数
```c++
#include <functional>

template <class T>
T GetAbsMinus(T Val1, T Val2)
{
	return abs(Val1 - Val2);
}

int main()
{
	std::function<int(int, int)> func1 = GetAbsMinus<int>;
	std::function<float(float, float)> func2 = GetAbsMinus<float>;
	std::cout << func1(15, 20) << std::endl; //5
	std::cout << func2(15.2, 20.5) << std::endl; //5.3
	return 0;
}
```

## 包装lambda函数
```c++
#include <functional>

auto GetAbsMinus = [](int nVal1, int nVal2)
	{return abs(nVal1 - nVal2); };

int main()
{
	std::function<unsigned int(int, int)> func = GetAbsMinus;
	std::cout << func(15.2, 20.5) << std::endl; //5
	return 0;
}
```

## 包装函数对象
```c++
#include <functional>

struct GetAbsMinus
{
	unsigned int operator() (int nVal1, int nVal2)
	{
		return abs(nVal1 - nVal2);
	}
};

int main()
{
	std::function<unsigned int(int, int)> func = GetAbsMinus();
	std::cout << func(15.2, 20.5) << std::endl; //5
	return 0;
}
```

## 包装类的静态成员函数
```c++
#include <functional>

class Human
{
public:
	static std::string GetName() { return strName; };
private:
	static std::string strName;
};

std::string Human::strName = "dzw";


int main()
{
	std::function<std::string()> func = Human::GetName;
	std::cout << func() << std::endl; //"dzw"
	return 0;
}
```

## 包装类的成员函数
需要配合`std::bind`使用；
```c++
#include <functional>

class Human
{
public:
	int GetAge(int nCurYear) { return nCurYear - nBirthYear; };
public:
	int nBirthYear;
};

int main()
{
	Human dzw;
	dzw.nBirthYear = 2000;
	std::function<int(int)> func = std::bind(&Human::GetAge, &dzw, std::placeholders::_1);
	std::cout << func(2022) << std::endl;

	return 0;
}
```


## 接受隐式转换

`std::function`支持参数和返回值的隐式类型转换，只要这些转换是安全的。这意味着，如果你有一个`std::function`对象，它的参数类型或返回类型与实际传递给它的函数、`Lambda`表达式、函数对象等的相应类型不完全匹配，但这些类型之间存在隐式转换，那么这个转换会自动发生；

这是`typedef`函数指针所不具备的灵活性；

```cpp
#include <functional>
#include <iostream>

void demonstrate(std::function<double(int)> func) {
    std::cout << func(5) << std::endl;
}

int main() {
    // Lambda表达式接受一个double，返回一个int
    auto lambda = [](double x) -> int {
        return static_cast<int>(x * x);
    };

    // 尽管std::function期望一个接受int返回double的函数，
    // 但由于存在隐式转换（int到double的参数转换，int到double的返回值转换），
    // 这里的lambda可以被正确调用。
    demonstrate(lambda);

    return 0;
}
```