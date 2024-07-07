在虚幻中，`TSubclassOf` 是一个模板类，用于表示某个类的子类类型。它允许你在代码中指定一个类的子类，并在运行时动态地使用这些子类。

# 主要用途

1. **类型安全**：`TSubclassOf` 提供了类型安全的方式来引用类的子类。
2. **蓝图支持**：在蓝图中，可以使用 `TSubclassOf` 来选择和指定类的子类。
3. **动态实例化**：可以在运行时动态地创建指定子类的实例。

# 示例代码

以下是一些使用 `TSubclassOf` 的示例代码：

```cpp
// 声明一个 TSubclassOf 变量
TSubclassOf<AActor> ActorSubclass;

// 在构造函数或其他地方初始化
ActorSubclass = AMyCustomActor::StaticClass();

// 动态创建实例
if (ActorSubclass)
{
    AActor* NewActor = GetWorld()->SpawnActor<AActor>(ActorSubclass);
}
```
