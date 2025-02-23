在虚幻引擎（Unreal Engine）中，`BlueprintLibrary` 是一种用于扩展蓝图（Blueprint）功能的类。它通常用于创建一组静态函数，这些函数可以在蓝图中调用。`BlueprintLibrary` 类的主要目的是提供一个可以在蓝图中使用的工具集，帮助开发者实现更复杂的逻辑，而不需要编写C++代码。

以下是一些关于 `BlueprintLibrary` 的关键点：

1. **静态函数**：`BlueprintLibrary` 中的函数通常是静态的，这意味着它们不需要实例化对象即可调用。

2. **BlueprintCallable**：这些函数通常标记为 `BlueprintCallable`，使得它们可以在蓝图中被调用。

3. **BlueprintPure**：如果一个函数不改变任何状态（即没有副作用），它可以被标记为 `BlueprintPure`，这意味着它可以在蓝图中作为纯函数使用；这意味着它们可以直接连接到其他节点的输入或输出，不需要执行线。

4. **扩展功能**：通过创建自定义的 `BlueprintLibrary`，开发者可以扩展引擎的功能，添加新的工具和实用程序，供蓝图使用。

5. **模块化**：`BlueprintLibrary` 可以被组织成模块，使得代码更易于管理和重用。

在C++中创建一个自定义的 `BlueprintLibrary` 类的基本步骤如下：

```cpp
// MyBlueprintLibrary.h
#pragma once

#include "Kismet/BlueprintFunctionLibrary.h"
#include "MyBlueprintLibrary.generated.h"

UCLASS()
class MYPROJECT_API UMyBlueprintLibrary : public UBlueprintFunctionLibrary
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintCallable, Category = "MyCategory")
    static void MyFunction();
};
```

```cpp
// MyBlueprintLibrary.cpp
#include "MyBlueprintLibrary.h"

void UMyBlueprintLibrary::MyFunction()
{
    // Function implementation
}
```

