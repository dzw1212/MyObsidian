在虚幻引擎（Unreal Engine）中，`UPROPERTY` 宏是用来自动处理C++类中成员变量的序列化、网络复制和编辑器集成的一种机制。这里是一些关于如何使用 `UPROPERTY` 的详细信息：

#  基本用法

`UPROPERTY` 宏可以应用于类的成员变量，以便它们与虚幻引擎的各种系统集成，如蓝图（Blueprints）、编辑器属性窗口等。基本语法如下：

```cpp
UPROPERTY([修饰符])
类型 变量名;
```

#  常用修饰符

在 Unreal Engine 中，`UPROPERTY` 是用于声明类成员变量的宏，它提供了许多选项来控制属性的行为、可见性和编辑权限。以下是一些常用的 `UPROPERTY` 选项：
## 访问权限

- **EditAnywhere**: 属性可以在编辑器中任何地方编辑。
- **EditDefaultsOnly**: 属性只能在类默认值中编辑，不能在实例中编辑。
- **EditInstanceOnly**: 属性只能在实例中编辑。
- **VisibleAnywhere**: 属性在编辑器中可见，但不可编辑。
- **VisibleDefaultsOnly**: 属性在类默认值中可见，但不可编辑。
- **VisibleInstanceOnly**: 属性在实例中可见，但不可编辑.

## 蓝图相关

- **BlueprintReadOnly**: 属性在蓝图中可读，但不可写。
- **BlueprintReadWrite**: 属性在蓝图中可读写。
- **BlueprintAssignable**: 适用于委托，允许在蓝图中绑定事件。
- **BlueprintCallable**: 允许在蓝图中调用函数。
- **BlueprintAuthorityOnly**: 仅在服务器上可用的蓝图功能。

## 复制和网络

- **Replicated**: 属性会在网络中复制。
- **ReplicatedUsing**: 指定一个函数，当属性被复制时调用。
- **Transient**: 属性不会被序列化或复制。

## 编辑器和开发

- **Category**: 指定属性在编辑器中显示的类别。
- **DisplayName**: 指定属性在编辑器中显示的名称。
- **ToolTip**: 为属性提供一个工具提示。
- **AdvancedDisplay**: 将属性放在高级属性部分。

## 元数据

- **meta**: 用于指定额外的元数据，如 `ClampMin`, `ClampMax`, `UIMin`, `UIMax`, `BindWidget`, `BindWidgetOptional` 等。

---


```cpp
UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Character Stats", meta = (ClampMin = "0.0", ClampMax = "100.0"))
float Health;

UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Components")
UStaticMeshComponent* MeshComponent;

UPROPERTY(ReplicatedUsing = OnRep_Health)
float Health;
```

这些选项可以组合使用，以满足特定的需求；


# 示例

```cpp
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyActor.generated.h"

UCLASS()
class MYGAME_API AMyActor : public AActor
{
    GENERATED_BODY()

public:
    // 可在编辑器和蓝图中读写的浮点数
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="My Category")
    float MyFloat;

    // 只在编辑器中可见的向量
    UPROPERTY(VisibleAnywhere, Category="My Category")
    FVector MyVector;

    // 不会被保存和加载的整数
    UPROPERTY(Transient)
    int32 MyTransientInt;
};
```

# 注意事项

- `UPROPERTY` 必须放在类的声明中，通常在头文件（.h）中。
- 使用 `UPROPERTY` 的变量必须是UObject的子类。
- 修饰符可以组合使用，以满足不同的需求。

