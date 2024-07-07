在虚幻引擎（Unreal Engine）中，`UENUM` 宏用于定义一个枚举类型，使其能够与引擎的元数据系统集成，从而支持蓝图等功能。以下是如何在虚幻引擎的C++中使用 `UENUM` 的一些详细步骤和示例：

# 基本语法

`UENUM()` 宏可以放在枚举定义之前，以便将枚举注册到虚幻的反射系统中。基本的语法结构如下：

```cpp
UENUM(BlueprintType)
enum class EMyEnumType : uint8
{
    Value1 UMETA(DisplayName = "Value One"),
    Value2 UMETA(DisplayName = "Value Two"),
    Value3 UMETA(DisplayName = "Value Three")
};
```

#  参数解释

- `BlueprintType`：这是一个常用的参数，它允许这个枚举在蓝图中被使用。
- `uint8`：枚举的底层类型，通常使用 `uint8` 来节省内存，但也可以根据需要使用其他整数类型。
- `UMETA(DisplayName = "Value One")`：这是一个元数据标记，用于定义在编辑器中显示的名称。

# 在类中使用 UENUM

你可以在任何类中定义和使用 `UENUM`，例如：

```cpp
#include "CoreMinimal.h"
#include "GameFramework/Actor.h"
#include "MyEnumActor.generated.h"

UENUM(BlueprintType)
enum class EActorState : uint8
{
    Idle UMETA(DisplayName = "Idle"),
    Moving UMETA(DisplayName = "Moving"),
    Attacking UMETA(DisplayName = "Attacking")
};

UCLASS()
class MYGAME_API AMyEnumActor : public AActor
{
    GENERATED_BODY()

public:
    // 使用枚举作为成员变量
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "State")
    TEnumAsByte<EActorState> CurrentState;
};
```
