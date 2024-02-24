# 头文件

## 自给自足

每个头文件都应该可以作为第一个头文件被`#include`，具有相对独立性；

头文件应该包含它所需要的其他头文件，如果你的头文件中用到了其他类或函数的声明，那么应该直接`#include`所需的其他头文件，而不是依赖于特定的包含顺序；

反例：

```cpp
//math_utils.h

//这个头文件忘记了包含 <vector>
std::vector<int> generatePrimes(int limit);
```

```cpp
//main.cpp

#include <vector> // 这里包含了<vector>，但是应该由math_utils.h自己包含
#include "math_utils.h" //能够通过编译，但是依赖了include顺序

int main() {
    auto primes = generatePrimes(100);
    for (int prime : primes) {
        std::cout << prime << " ";
    }
    return 0;
}
```

## \#define保护

所有头文件都应该有`#define`保护来防止头文件被多重包含，标准格式为`<PROJECT>_<PATH>_<FILE>_H_`；

比如`foo/src/bar/baz.h`的为：

```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_
...
#endif //FOO_BAR_BAZ_H_
```

### 与\#pragma once的区别

尽管`#pragma once`更为简洁，但是它没有完全取代`#define`保护：

1. `#pragma once`时一种编译器特定的指令，不是C++标准的一部分，尽管大部分编译器都支持它，但不能保证所有编译器的行为一致，因此移植性不能保证；
2. 在某些情况下（比如相同的文件通过不同路径被包含时），`#pragma once`可能无法正常工作；

## 前置声明

优点：
- 节省编译时间；
- 节省不必要的重新编译的时间（头文件的一点改动引起整个项目的重新编译）；

缺点：
- 只能用于指针、引用和函数的声明（占用内存大小确定），如果需要访问类的成员、继承类或创建类的实例，则必须包含相应头文件；
- 不利于后续维护，如果以前只需要指针但现在需要访问成员，则需要修改所有前置声明的地方；

建议还是优先使用`#include`；

## 内联函数

不要内联超过10行、或者包含循环/`switch`语句/递归的函数；

## \#include顺序

推荐顺序如下：
0. 自身对应的头文件；
1. C系统文件；
2. C++系统文件；
3. 其他库里的.h文件；
4. 本项目内的.h文件；
5. 平台特定的头文件；

比如`google-awesome-project/src/foo/internal/fooserver.cpp`的包含顺序如下：

```cpp
#include "foo/public/fooserver.h" // 优先位置

#include <sys/types.h>
#include <unistd.h>

#include <hash_map>
#include <vector>

#include "base/basictypes.h"
#include "base/commandlineflags.h"
#include "foo/public/bar.h"

#ifdef LANG_CXX11
#include <initializer_list>
#endif
```

# 作用域

## 命名空间

1. 禁止使用内联命名空间（inline namespace），内联命名空间会自动把内部的标识符放到外层作用域，增加了歧义；

```cpp
namespace X
{
	inline namespace Y
	{
		void foo();
	}
}
//这种情况下，X::Y::foo与Y::foo可相互替代
```

2. 禁止使用`using`直接引用某个命名空间，比如`using namespace std`，也会增加歧义；

3. 不要在命名空间`std`内声明任何东西，这会导致未定义的后果和移植的困难；

4. 不要在头文件内使用命名空间别名，这会影响到所有包含它的文件；

```cpp
//.h
namespace baz = foo::bar::baz;
```

5. 使用命名空间将整个代码封装起来；

```cpp
//.h中
namespace myspace
{

class MyClass
{
	void Foo();
};

}
```

```cpp
//.cpp中
namespace myspace
{

void MyClass::Foo() {...}

}
```

## 匿名命名空间与静态变量

匿名命名空间是没有名称的命名空间。在C++中，使用匿名命名空间可以将其成员限定在定义它的文件内部，从而实现文件内部链接性。这意味着，匿名命名空间中声明的变量、函数、类等在其他文件中是不可见的，类似于给全局静态变量或函数提供了一种更加优雅的替代方案；

```cpp
// 在文件A中
namespace {
    int internalFunction() { return 42; }
}

// internalFunction 在这个文件中可见，但在其他文件中不可见
```

在`cpp`文件中定义一个不需要被外部引用的变量时，可以使用匿名命名空间或声明为静态变量，但不要在头文件中这样做，这样就失去了原本的“仅本文件中使用”的功能；

## 裸全局函数

尽量不要使用裸的全局函数，将一系列函数直接置于命名空间中更好，避免污染全局空间；

## 局部变量

使用“初始化时声明”取代“初始化后再赋值”，以提高效率（默认构造+拷贝复制>拷贝构造）；

```cpp
//bad
int i;
i = f();

//good
int i = f();
```

```cpp
//bad
std::vector<int> v;
v.push_back(1);
v.push_back(2);

//good
std::vector<int> v = {1, 2};
```

仅用于`if/while/for`作用域内的局部变量应当在这些语句中声明，以明确限制其作用域；但是有一个例外是，如果这个局部变量是一个非内置类型，每次进入或退出作用域都需要承担构造或析构成本，那还是放在作用域之外声明比较好；

```cpp
while (const char* p = strchr(str, '/))
	str = p + 1;
```

```cpp
//bad
for (int i = 0; i < 100; ++i)
{
	Foo f;
	f.doSomething(i);
}

//good
Foo f;
for (int i = 0; i < 100; ++i)
{
	f.doSomething(i);
}
```