在虚幻引擎（Unreal Engine）中，多播委托（`Multicast Delegates`）是一种特殊类型的委托，用于向多个订阅者广播事件。在C++中，虚幻引擎提供了一些宏来帮助定义和使用多播委托。

# 定义多播委托

首先，你需要使用宏`DECLARE_DYNAMIC_MULTICAST_DELEGATE`（或其变体，如带参数的`DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam`等）来声明多播委托。例如：

```cpp
#include "CoreMinimal.h"
#include "UObject/NoExportTypes.h"
#include "MyClass.generated.h"

UCLASS()
class UMyClass : public UObject
{
    GENERATED_BODY()

public:
    // 声明一个无参数的多播委托
    DECLARE_DYNAMIC_MULTICAST_DELEGATE(FMyMulticastDelegate);

    // 声明一个带一个整数参数的多播委托
    DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMyMulticastDelegateInt, int, Value);

    // 委托实例
    FMyMulticastDelegate OnSomethingHappened;
    FMyMulticastDelegateInt OnNumberChanged;
};
```

# 绑定到多播委托

要绑定函数到多播委托，可以使用`AddDynamic`宏。这通常在某个类的成员函数中完成：

```cpp
void UMyOtherClass::BindDelegates(UMyClass* MyClassInstance)
{
    if (MyClassInstance)
    {
        MyClassInstance->OnSomethingHappened.AddDynamic(this, &UMyOtherClass::HandleSomethingHappened);
        MyClassInstance->OnNumberChanged.AddDynamic(this, &UMyOtherClass::HandleNumberChanged);
    }
}

void UMyOtherClass::HandleSomethingHappened()
{
    // 处理事件
}

void UMyOtherClass::HandleNumberChanged(int Value)
{
    // 处理事件
}
```

# 触发多播委托

要触发多播委托，简单地调用委托实例就像调用函数一样：

```cpp
void UMyClass::DoSomething()
{
    // 触发无参数的委托
    OnSomethingHappened.Broadcast();

    // 触发带参数的委托
    OnNumberChanged.Broadcast(42);
}
```
