在虚幻引擎（Unreal Engine）中，`FObjectInitializer` 是一个用于对象初始化的类，它在构造函数中传递，用于初始化那些需要额外信息才能正确初始化的对象。这个类提供了一系列方法来帮助设置对象的属性和子对象。

# 为什么需要FObjectInitializer

1. **生命周期管理**
`FObjectInitializer` 确保在对象的构造过程中正确地创建和初始化组件。它处理组件的生命周期，确保在适当的构造阶段分配和初始化这些组件，这有助于避免初始化顺序问题或潜在的内存管理错误。

2. **反射系统集成**
虚幻引擎的反射系统允许引擎在运行时识别和操作对象的属性和方法。使用 `FObjectInitializer::CreateDefaultSubobject` 方法创建的组件会自动注册到反射系统中，这使得它们可以在编辑器中被查看和修改，也支持序列化和网络复制等功能。

3. **默认子对象的安全创建**
`CreateDefaultSubobject` 方法不仅创建组件，还确保它们作为默认子对象（Default Subobjects）被正确标记和管理。这对于确保对象复制（如在编辑器中复制粘贴Actor）和蓝图派生类的行为正确非常重要。

4. **编辑器和蓝图兼容性**
使用 `FObjectInitializer` 创建的组件可以在Unreal编辑器中更容易地暴露和编辑，并且可以被蓝图继承和修改。这增加了游戏项目的灵活性和可扩展性。

# 基本使用

`FObjectInitializer` 通常在自定义类的构造函数中使用。例如，如果你有一个游戏中的角色类，你可能需要在构造函数中使用 `FObjectInitializer` 来初始化角色的某些组件或属性。

```cpp
#include "GameFramework/Character.h"
#include "Components/SkeletalMeshComponent.h"

class AMyCharacter : public ACharacter
{
public:
    AMyCharacter(const FObjectInitializer& ObjectInitializer)
        : Super(ObjectInitializer)
    {
        // 使用 ObjectInitializer 来初始化组件
        SkeletalMeshComponent = ObjectInitializer.CreateDefaultSubobject<USkeletalMeshComponent>(this, TEXT("SkeletalMeshComponent"));
        // 设置默认网格
        static ConstructorHelpers::FObjectFinder<USkeletalMesh> Mesh(TEXT("SkeletalMesh'/Game/Path/To/Mesh.Mesh'"));
        if (Mesh.Succeeded())
        {
            SkeletalMeshComponent->SetSkeletalMesh(Mesh.Object);
        }
    }
};
```

# 方法和功能

`FObjectInitializer` 提供了几个有用的方法，如下：

- **CreateDefaultSubobject**：这是最常用的方法之一，用于创建并初始化类的子对象。这些子对象通常是组件，如上面示例中的 `USkeletalMeshComponent`。

- **DoNotCreateDefaultSubobject**：这个方法用于阻止自动创建默认子对象，如果你不需要某个默认组件，可以使用这个方法。

# 什么时候需要FObjectInitializer

## 1. 什么时候用空参数构造函数 ？

**答案：95% 的日常开发情况。**

如果你只是在创建一个新的 Actor 或 Object，给它添加一些组件（比如 StaticMesh、Camera），或者初始化一些基本变量，直接使用无参构造函数即可。引擎在底层会自动帮你处理好 `FObjectInitializer`。

**示例：**

```cpp
// .h 文件
UCLASS()
class AMyActor : public AActor
{
    GENERATED_BODY()

public:
    AMyActor(); // 直接无参即可
};

// .cpp 文件
AMyActor::AMyActor()
{
    // 正常创建组件、初始化变量
    MyMesh = CreateDefaultSubobject<UStaticMeshComponent>(TEXT("MyMesh"));
}

```

---

## 2. 什么时候必须传入 `const FObjectInitializer&`？

当你继承了一个已经写好的父类（比如引擎自带的 `ACharacter` 或你团队其他人写的基类），并且你需要**修改或删除父类在构造函数中创建的组件**时，你就必须使用它。

它主要提供两个核心功能：

### A. 替换父类的默认组件 (`SetDefaultSubobjectClass`)

假设你继承了引擎的 `ACharacter`，引擎默认给它挂载了一个 `UCharacterMovementComponent`。但你的游戏需要特殊的移动逻辑，你写了一个 `UMyCustomMovementComponent`，你想用你的组件替换掉引擎自带的组件，且名字保持不变。

**示例代码：**

```cpp
// .h 文件
UCLASS()
class AMyCharacter : public ACharacter
{
    GENERATED_BODY()

public:
    // 必须声明带有 FObjectInitializer 的构造函数
    AMyCharacter(const FObjectInitializer& ObjectInitializer); 
};

// .cpp 文件
// 注意这里的写法：利用初始化列表 (Initializer List) 调用 Super（父类构造函数）时进行替换
AMyCharacter::AMyCharacter(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer.SetDefaultSubobjectClass<UMyCustomMovementComponent>(ACharacter::CharacterMovementComponentName))
{
    // 此时，你的角色使用的就是自定义的移动组件了
}

```

### B. 阻止创建父类的默认组件 (`DoNotCreateDefaultSubobject`)

假设你的父类创建了一个 `CameraComponent`，但你当前派生的这个子类是一个纯 AI 控制的敌人，不需要摄像机，你想省点性能。

**示例代码：**

```cpp
// .cpp 文件
AMyEnemyCharacter::AMyEnemyCharacter(const FObjectInitializer& ObjectInitializer)
    : Super(ObjectInitializer.DoNotCreateDefaultSubobject(TEXT("CameraComponentName")))
{
    // 这样，父类中名为 "CameraComponentName" 的组件就不会被实例化了
}

```

