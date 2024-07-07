*https://github.com/tranek/GASDocumentation*

`GAS`指的是`Gameplay Ability System`。这是一个高度可配置的框架，用于处理游戏中的技能、状态效果、属性（如生命值、法力值等）的管理。它是Unreal为了支持复杂的交互和游戏玩法逻辑而设计的，特别适用于需要复杂技能和状态管理的游戏，如RPG或MOBA游戏。

`GAS`的主要特点包括：
- **能力（Abilities）**：定义玩家或AI可以执行的动作，如攻击、治疗或施法。
- **效果（Gameplay Effects）**：用于修改属性（`Attributes`）的系统，可以表示增益、减益或其他状态改变。
- **属性（Attributes）**：游戏中的基础数值，如生命值、法力值等。
- **标签（Tags）**：一个灵活的系统，用于标记和分类各种游戏元素，以便于管理和引用。

`GAS`通过提供一套丰富的基础设施和工具，使得开发者能够构建复杂而灵活的游戏玩法逻辑，同时保持代码的可维护性和扩展性；

`GAS`系统已经过大型3A游戏的检验：《Paragon》，《Fortnite》；

# 启动GAS插件

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240421201649.png)

# 基础

为单人和多人游戏提供了一个开箱即用的解决方案，用于：

- 实现具有可选成本和冷却时间的基于等级的角色能力或技能（[[Gameplay Ability]]）
- 操作属于演员的数值属性（[[GamePlay Attribute]]）
- 将状态效果应用于演员（[[Gameplay Effect]]）
- 将GameplayTags应用于演员（[[Gameplay Tags]]）
- 生成视觉或声音效果（[[Gameplay Cues]]）

在多人游戏中，`GAS`支持以下客户端预测：

- 能力激活
- 播放动画蒙太奇
- 属性变化
- 应用`GameplayTags`
- 生成`GameplayCues`
- 通过与`CharacterMovementComponent`连接的`RootMotionSource`函数进行移动。

`GAS`必须在C++中设置，但设计师可以使用蓝图创建`GameplayAbilities`和`GameplayEffects`。

# 添加ASC

在`GAS`中，要实现技能（`Ability`）触发时来源显示对应的`Actor`，你需要确保每个参与技能交互的`Actor`都拥有一个[[ASC]]（`ASC`）。这个组件是`GAS`的核心，负责管理技能、属性和效果。以下是将`Ability System Component`附加到`Actor`并使用它的基本步骤：

## 1. 创建Ability System Component

首先，确保你的`Actor`类（可能是一个角色、怪物或任何其他游戏对象）在其类定义中包含了一个`Ability System Component`的引用。这通常在你的自定义`Actor`或`Character`的头文件中声明：

```cpp
#include "AbilitySystemComponent.h"
#include "AbilitySystemInterface.h"

UCLASS()
class YOURGAME_API AYourActor : public AActor, public IAbilitySystemInterface
{
    GENERATED_BODY()

public:
    AYourActor();

    // Ability System Component
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Abilities")
    UAbilitySystemComponent* AbilitySystemComponent;

    // 实现IAbilitySystemInterface中的方法
    virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;
};
```

```cpp
AYourActor::AYourActor()
{
    // 初始化Ability System Component
    AbilitySystemComponent = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("AbilitySystemComp"));
}

UAbilitySystemComponent* AYourActor::GetAbilitySystemComponent() const
{
    return AbilitySystemComponent;
}
```

## 2. 配置Ability System Component

在你的`Actor`初始化过程中（例如在构造函数或`BeginPlay`方法中），你可能需要配置`AbilitySystemComponent`，比如给它赋予初始的技能（`Abilities`）和属性（`Attributes`）。

## 3. 触发技能

当你想要触发一个技能时，你可以调用`AbilitySystemComponent`上的方法，如`TryActivateAbility`，并传入相应的技能标识符。这样，技能就会被触发，并且其来源会被正确地关联到拥有该`AbilitySystemComponent`的Actor上。

```cpp
AbilitySystemComponent->TryActivateAbility(AbilityToActivate, bActivateFromEvent);
```

## 4. 设置技能来源

在定义技能逻辑时（通常是在一个继承自`UGameplayAbility`的类中），你可以使用`AbilitySystemComponent`的所有者（即`Actor`）作为技能的来源。这样，当技能被触发时，你可以让技能效果（`Gameplay Effects`）或任何相关逻辑知道它们是由哪个`Actor`发起的。
