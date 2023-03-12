c++20新引入`std::format`用以格式化字符串，旨在替代之前的C风格的`sprintf`和C++风格的`stringstream`方法；

sprintf需要自己管理buf，stringstream拼接字符串既僵硬又繁琐；

std::format的原型为
```c++
template <typename ...Args>
std::string format(std::string_view fmt, const Args&... args);
```

具体使用方法参考：[[fmt格式]]