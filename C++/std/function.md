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
