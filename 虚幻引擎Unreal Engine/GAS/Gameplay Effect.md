在`GAS`中，`GameplayEffects`是用来表示对角色或对象状态的修改的一种机制。这些修改可以是临时的，比如加速或减速效果，也可以是永久的，比如增加角色的最大生命值。

`GameplayEffects`可以用来做很多事情，包括但不限于：

- 修改属性（`Attributes`），比如生命值、力量、速度等。
- 应用状态效果（`Status Effects`），比如中毒、燃烧、冰冻等。
- 触发游戏逻辑，比如当效果应用时播放动画或声音。
- 实现复杂的技能逻辑，比如根据目标当前的状态来决定效果的强度。

`GameplayEffects`通过`GameplayEffectSpecs`来定义，这些规格定义了效果的持续时间、强度以及它如何与其他效果叠加。它们可以被配置为即时生效的，持续一段时间的，或者是永久的，还可以设置为可以被清除或不可以被清除。

`GameplayEffects`是与`GameplayAbilities`和`GameplayAttributes`紧密相连的。`GameplayAbilities`通常是触发`GameplayEffects`的行为（例如，一个“火球”技能可能会应用一个燃烧效果），而`GameplayAttributes`则是`GameplayEffects`修改的属性（例如，生命值、魔法值等）。

---

# 触发GameplayEffect

首先，我们需要一个`GameplayAbility`来触发`GameplayEffect`。在这个例子中，我们将创建一个简单的能力，它会给目标应用一个`GameplayEffect`。

```cpp
// MyGameplayAbility.cpp

#include "AbilitySystemComponent.h"
#include "GameplayEffect.h"
#include "MyGameplayAbility.h"

void AMyGameplayAbility::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
    if (HasAuthority(&ActivationInfo))
    {
        // 假设我们有一个有效的GameplayEffect类引用
        TSubclassOf<UGameplayEffect> MyEffect = ...;

        // 创建GameplayEffect的实例
        FGameplayEffectSpecHandle EffectSpecHandle = MakeOutgoingGameplayEffectSpec(MyEffect, GetAbilityLevel());

        // 假设我们有一个目标
        FGameplayAbilityTargetDataHandle TargetData = ...;

        // 应用GameplayEffect到目标
        if (EffectSpecHandle.IsValid())
        {
            ActorInfo->AbilitySystemComponent->ApplyGameplayEffectSpecToTarget(*EffectSpecHandle.Data.Get(), TargetData);
        }
    }
}
```

# GameplayEffect修改GameplayAttribute

`GameplayEffect`通过其定义来修改`GameplayAttribute`。这通常在`GameplayEffect`的蓝图或C++类中定义。下面是一个C++中定义`GameplayEffect`以修改属性的例子。

首先，你需要定义你的`GameplayAttributes`。这通常在一个从`UGameplayAttributes`派生的类中完成。

```cpp
// MyGameplayAttributes.h

#include "CoreMinimal.h"
#include "AttributeSet.h"
#include "AbilitySystemComponent.h"
#include "MyGameplayAttributes.generated.h"

UCLASS()
class UMyGameplayAttributes : public UAttributeSet
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Attributes")
    FGameplayAttributeData Health;

    // 更多属性...
};
```

然后，在`GameplayEffect`的定义中，你可以指定它如何修改这些属性。这通常在`GameplayEffect`的蓝图中设置，但也可以通过代码来完成。以下是如何在C++中创建一个`GameplayEffect`的示例，该效果会修改上述`Health`属性。

```cpp
// 创建GameplayEffect的代码通常是在游戏启动时或者需要时进行，而不是在每次使用时都创建。

UGameplayEffect* CreateMyGameplayEffect()
{
    UGameplayEffect* NewEffect = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("MyHealthEffect")));
    if (NewEffect)
    {
        // 设置效果的持续时间类型
        NewEffect->DurationPolicy = EGameplayEffectDurationType::Instant;

        // 定义如何修改Health属性
        int32 Idx = NewEffect->Modifiers.Num();
        NewEffect->Modifiers.SetNum(Idx + 1);
        FGameplayModifierInfo& Modifier = NewEffect->Modifiers[Idx];
        Modifier.ModifierMagnitude = FScalableFloat(50.f); // 假设我们直接增加50点生命值
        Modifier.ModifierOp = EGameplayModOp::Additive;
        Modifier.Attribute = UMyGameplayAttributes::GetHealthAttribute();

        // 更多设置...
    }

    return NewEffect;
}
```

这些代码示例展示了创建和应用`GameplayEffect`的基本方法，以及如何通过这些效果来修改`GameplayAttributes`。实际项目中的实现可能会更复杂，包括处理网络同步、效果叠加、条件判断等。