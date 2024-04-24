在C++中，转型操作符用于在不同类型之间进行显式类型转换。
每种转型操作符都有其特定的用途和适用场景：

1. static_cast
	`static_cast`是最常用的转型操作符，用于基本数据类型之间的转换，以及向上（子类到基类）或向下（基类到子类，但不涉及多态）的类型转换。

```cpp
int i = 10;
double d = static_cast<double>(i); // 基本数据类型转换

class Base {};
class Derived : public Base {};
Base* b = new Derived;
Derived* d = static_cast<Derived*>(b); // 向下转型
```

2. const_cast
	`const_cast`用于修改类型的`const`或`volatile`属性。最常见的用途是去除对象的`const`属性，但要确保这样做是安全的，不会导致未定义行为。

```cpp
const int a = 10;
int* pa = const_cast<int*>(&a); // 去除const属性
```

3. dynamic_cast
	`dynamic_cast`主要用于处理多态，即在类的继承层次结构中安全地向下转型（基类指针转换为派生类指针）。它在运行时检查转换的有效性，如果转换不合法，则返回`nullptr`（对于指针）或抛出异常（对于引用）。

```cpp
class Base { virtual void dummy() {} };
class Derived : public Base { int a; };
Base* b = new Derived;
Derived* d = dynamic_cast<Derived*>(b); // 安全的向下转型
```

4. reinterpret_cast
	`reinterpret_cast`提供了一种低级别的转型能力，允许几乎任何指针类型转换为任何其他指针类型，甚至整数类型。它不检查语义，转换的安全性和合理性完全由程序员负责。

```cpp
long p = 0x12345678;
char* cp = reinterpret_cast<char*>(&p); // 将long*转换为char*
```

## static_cast与dynamic_cast的区别

既然`static_cast`和`dynamic_cast`都能将基类指针转为派生类，那区别是什么呢？

- `static_cast`在编译时执行转换，不进行运行时类型识别；
	适用于明确直到转换是安全的场合，比如你明确直到某个基类指针实际上指向某个派生类；
	如果实际上基类指针指向的不是你所认为的目标类型，则会导致未定义行为；

- `dynamic_cast`在运行时检查对象的类型信息（即运行时类型识别`RTTI`），如果转换不合法，则会失败；对于指针类型，失败时返回`nullptr`，对于引用类型，失败时抛出`std::bad_cast`异常；
	主要用于多态场合，当不确定基类指针究竟指向哪个派生类时，使用`dynamic_cast`可以安全地转换；
	相比`static_cast`，具有更高的性能开销；