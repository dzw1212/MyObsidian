在虚幻引擎（Unreal Engine）中，`UPROPERTY` 宏是用来自动处理C++类中属性的序列化、网络复制和编辑器集成的一种机制。这里是一些关于如何使用 `UPROPERTY` 的详细信息：

#  基本用法

`UPROPERTY` 宏可以应用于类的成员变量，以便它们与虚幻引擎的各种系统集成，如蓝图（Blueprints）、编辑器属性窗口等。基本语法如下：

```cpp
UPROPERTY([修饰符])
类型 变量名;
```

#  常用修饰符

- `EditAnywhere`: 允许在编辑器中的任何地方修改该属性。
- `BlueprintReadWrite`: 允许蓝图读取和修改该属性。
- `VisibleAnywhere`: 在编辑器中可见但不可修改。
- `Transient`: 不会被序列化，通常用于临时数据。
- `Category`: 后面跟着一个字符串，该字符串定义了属性在编辑器中显示的类别名称，可以自定义这个名称，以反映属性的用途或它们所属的逻辑组。
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

