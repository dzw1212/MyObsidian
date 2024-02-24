`explicit`关键词在 C++ 中用于**限制类构造函数的隐式自动转换**。默认情况下，如果一个类的构造函数只接受一个参数，那么编译器可以将该类型的对象隐式地从该参数类型转换。使用 `explicit`关键词可以防止这种隐式转换，要求必须显式地进行类型转换或对象的构造，==主要目的是为了提高代码的安全性和可读性，避免潜在的错误和不明确的行为==；

使用`explicit`关键词的主要目的是增加代码的清晰度和预防潜在的错误，特别是在类型转换时。它通常用于单参数构造函数和某些类型的转换操作符；

```cpp
class MyClass {
public:
    MyClass(int x) { /* ... */ }
};

void func(MyClass mc) { /* ... */ }

int main() {
    func(10); // 隐式调用 MyClass(int)，允许将 int 转换为 MyClass
}
```

```cpp
class MyClass {
public:
    explicit MyClass(int x) { /* ... */ }
};

void func(MyClass mc) { /* ... */ }

int main() {
    func(10); // 错误：不能隐式地从 int 转换为 MyClass
    func(MyClass(10)); // 正确：显式地构造 MyClass 对象
}
```