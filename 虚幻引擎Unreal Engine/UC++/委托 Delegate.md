# 单播

单一委托是最简单的委托类型，它允许绑定一个单一的函数；当委托被调用时，只有这个绑定的函数会被执行；

适用于需要简单回调机制的场景，比如按钮点击事件、简单的状态变化通知等；

## 定义

```cpp
DECLARE_DELEGATE(DelegateName);

DECLARE_DELEGATE_OneParam(DelegateName, ParamType); //带参数

DECLARE_DELEGATE_RetVal(ReturnType, DelegateName); //限制返回值类型
```

```cpp
class MyClass
{
public:
    FSimpleDelegate OnActionCompleted;
};
```

## 绑定

```cpp
void MyFunction()
{
    UE_LOG(LogTemp, Warning, TEXT("Action Completed!"));
}

void ExampleUsage()
{
    MyClass MyObject;
    MyObject.OnActionCompleted.BindStatic(&MyFunction);
}
```

## 调用

```cpp
void PerformAction()
{
	// 执行某些操作
	// ...

	// 操作完成后调用委托
	if (OnActionCompleted.IsBound())
	{
		OnActionCompleted.Execute();
	}
}
```

# 组播

多播委托（`Multicast Delegates`）是一种特殊类型的委托，用于向多个订阅者广播事件；

## 定义

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

## 绑定

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

## 调用

```cpp
void UMyClass::DoSomething()
{
    // 触发无参数的委托
    OnSomethingHappened.Broadcast();

    // 触发带参数的委托
    OnNumberChanged.Broadcast(42);
}
```


# 单播与组播的区别

- **绑定方式**：单播委托用 `Bind()`，多播委托用 `Add()`（需手动管理移除，避免重复绑定）；

-  **执行方式**：单播委托用 `Execute()`，多播委托用 `Broadcast()`（或`Broadcast()`安全的`IsBound()`检查）;

- **线程安全**：多播委托的 `Broadcast()` 是非线程安全的，需自行处理竞态条件;

# 绑定方式

虚幻引擎之所以提供这么多不同的绑定方式，核心原因只有一个：**内存管理（Memory Management）和对象生命周期（Object Lifetime）**；

当委托（Delegate）被触发时，它需要知道目标函数所属的对象**是否还活着**。如果对象已经被销毁，而委托仍然尝试调用它的函数，就会导致程序崩溃（Crash）。

## 不同绑定方式详解

### `BindUObject` (最常用，最推荐)

* **适用对象：** 继承自 `UObject` 的类（如 `AActor`, `UActorComponent`, `UUserWidget` 等）。
* **机制：** 它是**弱引用（Weak Reference）** 的。在调用前，委托系统会自动检查该 UObject 是否有效（`IsValid`）或是否被标记为待销毁（Pending Kill）。
* **安全性：** **非常安全**。如果对象被 GC（垃圾回收）了，委托不会执行，也不会崩溃。
* **场景：** 任何涉及游戏逻辑、Actor 交互、UI 响应的场景。

### `BindSP` (Bind Shared Pointer)

* **适用对象：** 使用虚幻智能指针 `TSharedPtr` 管理的原生 C++ 类（非 `UObject`）。
* **要求：** 你的类通常需要继承 `TSharedFromThis<YourClass>`。
* **机制：** 它是**弱引用**的。它会持有一个 `TWeakPtr`，在执行前检查对象是否存在。
* **安全性：** **非常安全**。如果共享指针的引用计数归零，对象被销毁，委托自动失效。
* **场景：** Slate 界面编程、编辑器工具开发、自定义的 C++ 子系统（不继承 UObject 的纯逻辑类）。

### `BindRaw` (最危险，需谨慎)

* **适用对象：** 普通的原生 C++ 指针（Raw Pointer），且该对象**不是**由 `TSharedPtr` 管理的。
* **机制：** 它是**强行绑定**。委托系统**不会**（也无法）检查指针指向的内存是否还合法。
* **安全性：** **不安全**。如果对象被删除了，但委托没有解绑，触发时会直接导致 **Crash (Access Violation)**。
* **场景：**
	* 引用第三方库的静态对象。
	* 非常底层的系统，你有 100% 的信心能手动管理生命周期（即：在对象析构函数中必须调用 `Unbind`）。
* **一般不建议初学者使用。**

### `BindStatic`

* **适用对象：** 全局静态函数（Global Functions）或类的静态成员函数（Static Member Functions）。
* **机制：** 绑定函数地址。
* **安全性：** **安全**。因为静态函数一直存在于内存中，不存在“对象销毁”的问题。
* **场景：** 绑定到工具函数库、数学计算函数等不需要对象实例的方法。

### `BindLambda`

* **适用对象：** C++ Lambda 表达式（匿名函数）。
* **机制：** 直接执行一段临时的代码块。
* **安全性：** **取决于捕获列表（Capture List）**。
* 如果捕获了 `[this]` 或 `[&]`，而执行时对象已经销毁，会 **Crash**。
* 建议配合 `BindWeakLambda`（如果是 UObject）使用，或者谨慎处理捕获变量。
* **场景：** 逻辑非常简单（只有一两行代码），不想专门写一个成员函数时使用。

### `BindWeakLambda`

* 这是 `BindLambda` 的升级版，专门配合 `UObject` 使用。
* 语法：`BindWeakLambda(ContextObject, [this](){ ... })`。
* 它会在执行 Lambda 前检查 `ContextObject` 是否还活着。如果对象死了，Lambda 不会执行。**强烈推荐用这个替代普通的 BindLambda 来处理 UObject。**

### `BindDynamic` (仅限动态委托)

* **必须**用于 `DECLARE_DYNAMIC_...` 定义的委托。
* 允许序列化，允许在蓝图（Blueprint）中绑定。
* 函数必须标记为 `UFUNCTION()`。

## 对比

| 绑定方式 | 目标对象类型 | 安全性 (对象销毁后) | 是否持有引用 | 典型应用场景 |
| --- | --- | --- | --- | --- |
| **BindUObject** | `UObject` 派生类 | **安全** (不执行) | 否 (弱引用) | Actor, Component, Gameplay |
| **BindSP** | `TSharedPtr` 管理的类 | **安全** (不执行) | 否 (弱引用) | Slate UI, 自定义 C++ 工具类 |
| **BindRaw** | 普通 C++ 指针 | **危险** (崩溃) | 否 (裸指针) | 第三方库, 必须手动解绑 |
| **BindStatic** | 静态函数 | **安全** | N/A | 工具函数, 全局回调 |
| **BindLambda** | Lambda 表达式 | **视情况而定** | 取决于捕获 | 临时逻辑, 简单胶水代码 |
# UDELEGATE标识符

简单来说，**`UDELEGATE(...)` 之于委托（Delegate），就好比 `UCLASS(...)` 之于类，或者 `USTRUCT(...)` 之于结构体。**

在虚幻引擎中，当你写一个动态委托声明时：

```cpp
// 平时我们这样写
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FMySignature, float, Value);

```

虚幻头文件工具（UHT）非常聪明，它看到 `DECLARE_DYNAMIC...` 开头的宏，就会自动把它识别为反射类型，并在后台自动生成类似 `UDELEGATE` 的处理逻辑。**默认情况下，你不需要手动加 `UDELEGATE()` 宏，它也能工作。**

## 作用

最（也是几乎唯一）常见的场景：`BlueprintAuthorityOnly`

如果你定义了一个委托类型，并且你希望**只能在服务器端（Authority）** 调用或绑定这个委托，你需要这样写：

```cpp
// 显式使用 UDELEGATE 宏来添加限制
UDELEGATE(BlueprintAuthorityOnly)
DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnServerEvent, int32, PlayerID);

```

**效果：**
当你在蓝图中使用这个 `FOnServerEvent` 类型的委托时，如果蓝图节点试图在客户端上下文中调用它，编辑器可能会发出警告或限制，表明这个事件设计上只应该由服务器触发。
