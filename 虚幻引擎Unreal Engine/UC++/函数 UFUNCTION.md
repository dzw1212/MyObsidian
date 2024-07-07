在虚幻引擎（Unreal Engine）中，`UFUNCTION` 宏是用来修饰函数，使得这些函数能够利用虚幻引擎的特定功能，如蓝图调用、网络复制等。以下是一些常用的 `UFUNCTION` 用法：

#  基本用法

在C++中定义一个函数，你可以使用 `UFUNCTION()` 宏来修饰这个函数，使其可以在蓝图中调用。

```cpp
class AMyActor : public AActor
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category="My Functions")
    void MyFunction();
};
```

# 修饰符
## 蓝图交互

- **BlueprintCallable** - 允许蓝图调用这个函数。
- **BlueprintImplementableEvent** - 在C++中声明，由蓝图实现。
- **BlueprintNativeEvent** - 在C++中提供默认实现，蓝图可以重写。
## 网络

- **Server** - 标记为只在服务器上运行的函数。
- **Client** - 标记为只在客户端上运行的函数。
- **NetMulticast** - 在服务器和所有客户端上运行。
## 其他

- **Category** - 定义在蓝图编辑器中显示的类别。
- **DefaultToSelf** - 默认目标对象为调用者。
- **BlueprintPure** - 不改变对象状态的函数，不需要执行节点。

# 示例

```cpp
class AMyActor : public AActor
{
    GENERATED_BODY()

public:
    UFUNCTION(Server, Reliable, WithValidation)
    void ServerDoSomething();
    void ServerDoSomething_Implementation();
    bool ServerDoSomething_Validate();
};
```

在这个例子中，`ServerDoSomething` 函数被标记为服务器函数，必须在服务器上调用，并且具有验证逻辑来确保调用是有效的。

# 注意事项

- 使用 `UFUNCTION` 宏时，确保包含头文件 `UObject/Class.h`。
- 函数的实现应遵循虚幻的命名和编码规范，例如，对于 `BlueprintNativeEvent`，实现应该是 `FunctionName_Implementation`。
