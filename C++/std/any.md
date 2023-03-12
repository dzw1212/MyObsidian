*any作为一个可以存放不同类型的值的容器，可以避免在编程时因类型不能同时兼容声明太多的临时局部变量*


# Quick Start

```c++
#include <any>
#include <iostream>
 
int main()
{
    std::cout << std::boolalpha;
 
    // any type
    std::any a = 1;
    std::cout << a.type().name() << ": " << std::any_cast<int>(a) << '\n';
    a = 3.14;
    std::cout << a.type().name() << ": " << std::any_cast<double>(a) << '\n';
    a = true;
    std::cout << a.type().name() << ": " << std::any_cast<bool>(a) << '\n';

	a.type().name(); //"bool"
	a.type() == typeid(bool); //true
	
    // bad cast
    try
    {
        a = 1;
        std::cout << std::any_cast<float>(a) << '\n';
    }
    catch (const std::bad_any_cast& e)
    {
        std::cout << e.what() << '\n';
    }
 
    // has value
    a = 2;
    if (a.has_value())
    {
        std::cout << a.type().name() << ": " << std::any_cast<int>(a) << '\n';
    }
 
    // reset
    a.reset();
    if (!a.has_value())
    {
        std::cout << "no value\n";
    }
 
    // pointer to contained data
    a = 3;
    int* i = std::any_cast<int>(&a);
    std::cout << *i << "\n";
}
```

```
int: 1
double: 3.14
bool: true
bad_any_cast
int: 2
no value
3
```

---

any中的值是使用衰减类型存储的（比如数组的话存储为头指针），对于字符串常量，值的类型是`const char*`而不是`std::string`；
```c++
std::any a = "hello";

a.type() == typeid(std::string); //false
a.type() == typeid(const char*); //true
```

any没有定义比较运算符，没有hash函数，也没有取值函数，其值只有在运行时才直到类型；

一个有趣的用法是，any可以作为标准库容器的元素：
```c++
std::vector<std::any> vec;
vec.push_back(10);
vec.push_back(20.0);
vec.push_back("30");
```

# 构造

```c++
std::any a1; //empty
std::any a2 = 10; //类型为int
std::any a3 = "10"; //类型为const char*
std::any a4 = 10.0; //类型为double

//若要保存与推断值不同的类型，需要借助in_type_type或者make_any
std::any a5{std::in_place_type<unsigned char>, 10 };
std::cout << a5.type().name() << std::endl; //"unsigned char"

a5 = std::make_any<float>(20);
std::cout << a5.type().name() << std::endl; //"float"
std::cout << std::any_cast<float>(a5) << std::endl; //20

//若是通过多个参数初始化，则需要创建该复杂对象或者使用in_place_type声明类型
std::any a6{std::complex{3.0, 4.0}};
std::any a7{std::in_place_type<std::complex<``double``>>, 3.0, 4.0};
```

# 赋值

直接使用`=`或者使用`emplace`；

```c++
std::any a;
a = 42; // a contains value of type int
a = "hello"; // a contains value of type const char*

a.emplace<std::string>("hello world"); // a contains value of type std::string
```