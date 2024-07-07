在虚幻中，`FName` 是一种轻量级、高效的字符串类型，主要用于标识和查找对象。与 `FString` 不同，`FName` 具有更高的性能，特别是在需要频繁比较和查找的场景中。

`FName` 常用于标识对象的名称，如组件、资源、变量等。
1. **标识对象**：`FName` 常用于标识对象的名称，如组件、资源、变量等。
2. **高效比较**：`FName` 的比较操作非常高效，因为它们在内部使用了哈希表。
3. **查找和引用**：`FName` 可以用于查找和引用对象，特别是在反射系统中。
4. **避免重复字符串**：`FName` 在内部使用了字符串表，避免了重复字符串的内存开销。


```cpp
// 创建 FName 对象
FName MyName("MyActorName");

// 比较 FName 对象
if (MyName == FName("MyActorName"))
{
    // 名称匹配
}

// 使用 FName 查找对象
AActor* FoundActor = FindObject<AActor>(ANY_PACKAGE, *MyName.ToString());
```

