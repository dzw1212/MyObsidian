在`GAS`中，`GameplayEffects`是用来表示对角色或对象状态的修改的一种机制。这些修改可以是临时的，比如加速或减速效果，也可以是永久的，比如增加角色的最大生命值。

`GameplayEffects`可以用来做很多事情，包括但不限于：

- 修改属性（`Attributes`），比如生命值、力量、速度等。
- 应用状态效果（`Status Effects`），比如中毒、燃烧、冰冻等。
- 触发游戏逻辑，比如当效果应用时播放动画或声音。
- 实现复杂的技能逻辑，比如根据目标当前的状态来决定效果的强度。

`GameplayEffects`通过`GameplayEffectSpecs`来定义，这些规格定义了效果的持续时间、强度以及它如何与其他效果叠加。它们可以被配置为即时生效的，持续一段时间的，或者是永久的，还可以设置为可以被清除或不可以被清除。

`GameplayEffects`是与`GameplayAbilities`和`GameplayAttributes`紧密相连的。`GameplayAbilities`通常是触发`GameplayEffects`的行为（例如，一个“火球”技能可能会应用一个燃烧效果），而`GameplayAttributes`则是`GameplayEffects`修改的属性（例如，生命值、魔法值等）。

FYI：`UGameplayEffect`是一个纯数据类，不应在其中添加任何的逻辑；

# 分类

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240722011413.png)


根据效果的持续时间和触发方式，`GameplayEffect`可以被分为三种主要类型：瞬时的（`Instant`）、持续的（`Duration`）、和周期的（`Periodic`）：

1. 瞬时的（`Instant`）
	一次性的，它们在被应用时立即发生并完成。这种效果通常用于立即改变目标的属性值，如立即恢复一定量的生命值或立即增加力量。瞬时效果不会有持续时间，它们的改变通常是永久性的，直到被其他效果修改；

2. 持续的（`Duration`）
	在一段时间内有效，它们在应用时开始，并在预定的时间后结束。这期间，效果会持续影响目标。例如，一个增加防御力的持续效果可能在10秒内有效，期间防御力提高，效果结束后防御力恢复正常。持续效果适用于需要临时改变属性的情况；
	
3. 周期的（`Periodic`）
	定期触发的效果。它们在一定周期内重复触发，每次触发都像瞬时效果一样立即发生。周期效果可以用于模拟如每秒恢复生命值这样的效果。周期效果通常有一个开始和结束时间，效果在这段时间内周期性地发生；

# GE实例

`GameplayEffect`通常不会被实例化，当一个`GameplayAbility`或`ASC`想要应用一个`GameplayEffect`时，会创建一个`GameplayEffectSpec`，成功应用的话该`spec`会被添加到名为`FActiveGameplayEffect`的结构中，`ASC`会在`ActiveGameplayEffects`中跟踪该结构；

## 实例上下文

`GameplayEffectContext`保存`spec`的发起者`instigator`和目标数据`TargetData`，可以方便地在`ModifierMagnitudeCalculations`、`GameplayEffectExecutionCalculations`、`AttributeSets`和`GameplayCues`等场景之间传递任意数据；

子类化一个`GameplayEffectContex`的步骤：
1. 创建子类；
2. 覆写`FGameplayEffectContext::GetScriptStruct`；
3. 覆写`FGameplayEffectContext::Duplicate`；
4. 如果需要复制，则覆写`FGameplayEffectContext::NetSerialize`；
5. 实现`TStructOpsTypeTraits`；
6. 在`AbilitySystemGlobals`类中覆写`AllocGameplayEffectContext`以返回一个子类的新对象；
## 查看实例创建者

`GameplayEffectContextHandle` 是用来封装和传递关于一个 `GameplayEffect` 的上下文信息的。这包括了哪个角色或对象是这个效果的来源。要获取创建了 `GameplayEffectSpec` 的实体，可以使用 `GameplayEffectContextHandle` 提供的方法来获取这些信息；

```cpp
// 假设你有一个 GameplayEffectSpec 对象名为 Spec
GameplayEffectContextHandle ContextHandle = Spec.GetContext();

// 现在你可以使用 ContextHandle 来访问上下文信息，例如获取触发者
if (ContextHandle.IsValid())
{
    AActor* InstigatorActor = ContextHandle.GetInstigator();
    if (InstigatorActor)
    {
        // 使用 InstigatorActor
    }
}
```

# 应用GE

有多种方法来应用`GameplayEffect`，通常接口会使用`ApplyGameplayEffectToXXX`的形式，这些接口最终都会调用目标身上的`UAbilitySystemComponent::ApplyGameplayEffectSpecToSelf`函数；

# 移除GE

同样的，有很多方法来移除`GameplayEffect`，通常接口采用`RemoveActiveGameplayEffect`的形式，最终都会调用目标身上的`FActiveGameplayEffectsContainer::RemoveActiveEffects`函数；

# 监听GE

```cpp
//监听应用
virtual void OnActiveGameplayEffectAddedCallback(UAbilitySystemComponent* Target, const FGameplayEffectSpec& SpecApplied, FActiveGameplayEffectHandle ActiveHandle);

AbilitySystemComponent->OnActiveGameplayEffectAddedDelegateToSelf.AddUObject(this, &APACharacterBase::OnActiveGameplayEffectAddedCallback);

//监听移除
virtual void OnRemoveGameplayEffectCallback(const FActiveGameplayEffect& EffectRemoved);

AbilitySystemComponent->OnAnyGameplayEffectRemovedDelegate().AddUObject(this, &APACharacterBase::OnRemoveGameplayEffectCallback);
```

# GE如何修改属性

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

# GE中的标签

`GameplayEffect`中包含多个`GameplayTagCOntainer`，比如有负责`Added`的标签容器，有负责`Removed`的标签容器，有组合而成的`Combined`标签容器；

## 标签类型

1. **Gameplay Effect Asset Tags**:
   - 这些标签附加在 `GameplayEffect` 资产本身上，用于描述和识别该效果的一般属性或类别。它们不直接影响效果的逻辑执行，但可以用于筛选和识别特定的效果。

2. **Granted Tags**:
   - 当 `GameplayEffect` 被应用时，它会将这些标签授予目标。这意味着这些标签会被添加到目标对象的标签集合中，可以用来触发其他效果或改变游戏逻辑的行为。

3. **Ongoing Tag Requirements**:
   - 这些是 `GameplayEffect` 持续有效所需满足的标签条件。如果这些条件在效果应用后不再满足，效果可能会被终止。这些标签要求确保效果只在特定的游戏状态下活跃。

4. **Application Tag Requirements**:
   - 这些标签定义了 `GameplayEffect` 能够被应用到目标上的前提条件。如果目标不满足这些标签要求，那么效果将不会被应用。这用于控制效果的适用性和限制。

5. **Remove Gameplay Effects with Tags**:
   - 这个功能允许 `GameplayEffect` 在应用时移除具有特定标签的其他效果。这是一种管理或清理目标上已存在效果的方式，常用于取消或替换效果。


# GE中的修改器 Modifiers

修改器可以改变属性，一个`GameplayEffect`可以有0或多个修改器，每个修改器只负责通过一个指定的操作改变一个属性；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240722012427.png)

## 修改器操作 Modifier Op

修改器的操作有：`Add`，`Multiply`，`Divide`，`Override`；
其聚合公式在`GameplayEffectAggregator.cpp`中的`FAggregatorModChannel::EvaluateWithBase`定义如下：
```cpp
((InlineBaseValue + Additive) * Multiplicitive) / Division
```

属性的当前值是所有修改器与其基础值相作用的总和；
[[GamePlay Attribute#基础值与当前值]]

### 覆盖逻辑

任意覆盖操作的修改器都会覆盖最终值，以最后一个应用的覆盖类修改器为准；

### 乘除逻辑

比如说基础值为100，有两个系数为1.5的乘法修改器，其结果并不是$100*1.5*1.5=225$，而是$100+(100*0.5)+(100*0.5)=200$；

假如有三个乘法修改器，系数分别为$a,b,c$，则最终系数为
$$
\lambda=1+(a-1)+(b-1)+(c-1)
$$
这与常规的逻辑有所区别；

---

如果想要修改计算逻辑，可以修改`FAggregatorModChannel::EvaluateWithBase`：
```cpp
float FAggregatorModChannel::EvaluateWithBase(float InlineBaseValue, const FAggregatorEvaluateParameters& Parameters) const
{
	...
	float Multiplicitive = MultiplyMods(Mods[EGameplayModOp::Multiplicitive], Parameters);
	float Division = MultiplyMods(Mods[EGameplayModOp::Division], Parameters);
	...

	return ((InlineBaseValue + Additive) * Multiplicitive) / Division;
}

float FAggregatorModChannel::MultiplyMods(const TArray<FAggregatorMod>& InMods, const FAggregatorEvaluateParameters& Parameters)
{
	float Multiplier = 1.0f;

	for (const FAggregatorMod& Mod : InMods)
	{
		if (Mod.Qualifies())
		{
			Multiplier *= Mod.EvaluatedMagnitude;
		}
	}

	return Multiplier;
}
```

## 修改器幅度类型 Modifier Magnitude

- **Scalable Float**
	`FScalableFloats` 是一种可以指向数据表的结构，该数据表以变量为行，以等级为列。可缩放浮点数会自动读取指定表格行在能力当前等级（或不同等级，如果在 `GameplayEffectSpec` 中被覆盖）下的值。该值还可以通过系数进一步操作。如果没有指定数据表/行，则会将该值视为 1，因此可以使用系数在所有级别硬编码一个值；

- **Attribute Based**
	基于属性的修改器获取源（创建 `GameplayEffectSpec` 的用户）或目标（接收 `GameplayEffectSpec` 的用户）上的支持属性的当前值或基准值，并通过系数和前后系数添加对其进行进一步修改。快照是指在创建 `GameplayEffectSpec` 时捕获后备属性，而不快照是指在应用 `GameplayEffectSpec` 时捕获属性；

- **Custom Calculation Class**
	自定义计算类为复杂的修改器提供了最大的灵活性。该修改器使用 `ModifierMagnitudeCalculation` 类，并可通过系数、系数前添加和系数后添加进一步处理生成的浮点数值；

- **Set By Caller**
	`SetByCaller` 修改器是在运行时由能力或在 `GameplayEffectSpec` 上制作 `GameplayEffectSpec` 的人员在 `GameplayEffect` 外部设置的值。例如，如果要根据玩家按住按钮的时间来设置伤害，就需要使用 `SetByCaller`。`SetByCaller` 本质上是 `TMap<FGameplayTag,float>`，它存在于 `GameplayEffectSpec` 中。修改器只是告诉聚合器查找与所提供的 `GameplayTag` 相关联的 `SetByCaller` 值。修改器使用的 `SetByCaller` 只能使用 `GameplayTag` 版本的概念。此处禁用 `FName` 版本。如果修改器被设置为 `SetByCaller`，但 `GameplayEffectSpec` 上不存在具有正确 `GameplayTag` 的 `SetByCaller`，游戏将抛出运行时错误并返回 0 值；


```cpp
#include "GameplayEffect.h"
#include "GameplayModCalculation.h"

// 自定义计算类
class UMyCustomCalculation : public UGameplayModCalculation
{
public:
    virtual float CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec& Spec) const override
    {
        // 自定义逻辑，返回计算结果
        return 100.0f;  // 示例返回值
    }
};

// 创建一个新的 Gameplay Effect 类
class UMyGameplayEffect : public UGameplayEffect
{
public:
    UMyGameplayEffect()
    {
        DurationPolicy = EGameplayEffectDurationType::Instant;

        // 使用 ScalableFloat
        {
            FGameplayModifierInfo ModifierInfo;
            ModifierInfo.Attribute = UAttributeSet::Health;
            ModifierInfo.ModifierOp = EGameplayModOp::Additive;
            ModifierInfo.ModifierMagnitude = FScalableFloat(50.f);
            Modifiers.Add(ModifierInfo);
        }

        // 使用 AttributeBased
        {
            FGameplayModifierInfo ModifierInfo;
            ModifierInfo.Attribute = UAttributeSet::Damage;
            ModifierInfo.ModifierOp = EGameplayModOp::Additive;
            FAttributeBasedModifier AttributeBasedModifier;
            AttributeBasedModifier.AttributeToCapture = UAttributeSet::Health;
            AttributeBasedModifier.Coefficient = 0.1f;
            ModifierInfo.ModifierMagnitude = FGameplayEffectModifierMagnitude(AttributeBasedModifier);
            Modifiers.Add(ModifierInfo);
        }

        // 使用 CustomCalculationClass
        {
            FGameplayModifierInfo ModifierInfo;
            ModifierInfo.Attribute = UAttributeSet::Speed;
            ModifierInfo.ModifierOp = EGameplayModOp::Multiplicative;
            ModifierInfo.ModifierMagnitude = FGameplayEffectModifierMagnitude(UMyCustomCalculation::StaticClass());
            Modifiers.Add(ModifierInfo);
        }

        // 使用 SetByCaller
        {
            FGameplayModifierInfo ModifierInfo;
            ModifierInfo.Attribute = UAttributeSet::Mana;
            ModifierInfo.ModifierOp = EGameplayModOp::Additive;
            ModifierInfo.ModifierMagnitude = FGameplayEffectModifierMagnitude(FGameplayTag::RequestGameplayTag(FName("Skill.ManaCost")));
            Modifiers.Add(ModifierInfo);
        }
    }
};
```

## 修改器标签

每个修改器都可以设置源标签`SourceTags`和目标标签`TargetTags`，这些标签主要起到筛选的作用，在`GameplayEffect`应用时，可以通过`SourceTagFilter`和`TargetTagFilter`来筛选修改器；

因此对于周期性的`GameplayEffect`，只需要在第一次应用时，设置源标签和目标标签即可；

# 可堆叠的GE

一般来说，每次应用`GameplayEffect`时，都会创建一个`GameplayEffectSpec`的新实例；如果不想每次都创建重复的实例，可以将`GameplayEffect`设置为“堆叠`Stack`”，这样当应用时原先存在的实例的堆叠次数`stack count`会增加；

堆叠一般用于持续性的或无限时间的`GameplayEffect`；

## 堆叠类型

- 按源聚合 **Aggregator by Source**
	来自同一来源的效果会堆叠在一起；
	目标上的每个源 `ASC` 都有一个单独的堆叠实例，每个源可应用 X 数量的堆叠；
	例如，如果一个角色多次施放同一个增益效果，这些效果会根据来源进行聚合堆叠；

-  按目标聚合 **Aggregator by Target**
	效果会根据目标进行堆叠，不论效果的来源是谁；
	无论源是什么，目标上都只有一个堆栈实例，但不得超过共享堆栈限制；

# 赋予新能力的GE

`GameplayEffect`可以为`ASC`赋予新的`GameplayAbility`，但仅限持续性的和无限时间的`GameplayEffect`；

```cpp
#include "GameplayEffect.h"
#include "Abilities/GameplayAbility.h"
#include "GameplayEffectExecutionCalculation.h"
#include "AbilitySystemComponent.h"

// 定义一个新的 GameplayEffect 类
UCLASS()
class MYGAME_API UMyGameplayEffect : public UGameplayEffect
{
    GENERATED_BODY()

public:
    UMyGameplayEffect()
    {
        // 在构造函数中设置 GameplayEffect 的属性
        DurationPolicy = EGameplayEffectDurationType::Instant;

		// 设置移除时的行为
		        OnEffectRemovedPolicy = EGameplayEffectRemovalPolicy::RemoveWhenDurationExpired;

        // 假设 UMyAbility 是一个已经定义的 GameplayAbility 类
        if (AbilityToAdd)
        {
            // 创建一个 GameplayAbilitySpec，赋予特定的 Ability
            FGameplayAbilitySpec AbilitySpec(AbilityToAdd, 1, INDEX_NONE, this);
            GrantedAbilities.Add(AbilitySpec);
        }
    }

    // 指定要赋予的 Ability 类
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category = "Abilities")
    TSubclassOf<UGameplayAbility> AbilityToAdd;
};
```

## 能力移除策略

- 立即取消 **Cancel Ability Immediately**
	当授予该能力的`GameplayEffect`移除时，对应的能力立即被移除；

- 结束时移除 **Remove Ability on End**
	授予的能力结束时，自动被移除；

- 不移除 **Do Nothing**
	授予的能力不会被自动移除，相当于永久拥有该能力，直到被手动移除；

# 作为能力成本的GE

`GameplayAbility`中有一个可选的`GameplayEffect`，专门用来作为使用该能力的成本 —— 即`ASC`需要拥有多少属性才能激活该能力，如果成本不足，则无法激活能力；

这个`Cost GameplayEffect`一般是即时的，带有一个或多个减少属性的修改器；

一种推荐的做法是，为多个需要成本的能力创建并公用一个`Cost GameplayEffect`，然后根据能力的不同需求创建不同的`spec`；

以下有两种实现方式：
1. 通过`MMC`，从`GameplayAbility`的实例中读取能力成本；
```cpp
float UPGMMC_HeroAbilityCost::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec & Spec) const
{
	const UPGGameplayAbility* Ability = Cast<UPGGameplayAbility>(Spec.GetContext().GetAbilityInstance_NotReplicated());

	if (!Ability)
	{
		return 0.0f;
	}

	return Ability->Cost.GetValueAtLevel(Ability->GetAbilityLevel());
}
```

2. 覆写`UGameplayAbility::GetCostGameplayEffect`，在该函数运行时读取自身的成本值并创建一个`GameplayEffect`；

# 作为冷却时间的GE

`GameplayAbility`有一个可选的`GameplayEffect`，专门用作该能力的冷却时间；如果能力处于冷却时间中，则无法激活；

这个`GE`是一个持续性的效果，没有修改器；

不推荐使用检测冷却时间`GE`是否存在来实现冷却（为什么？），推荐的做法是为冷却时间`GE`的`GrantedTags`中添加一个唯一的冷却标签`Cooldown Tag`，当能力激活+冷却时间`GE`应用到角色身上时，会自动将冷却标签添加到角色的`GameplayTagContainer`中；

同样的类似于“作为能力成本的`GE`”，推荐的做法是多个`GA`重用同一个冷却时间`GE`，以下有两种实现方式：

1. 使用`SetByCaller`；
```cpp
UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FScalableFloat CooldownDuration;

UPROPERTY(BlueprintReadOnly, EditAnywhere, Category = "Cooldown")
FGameplayTagContainer CooldownTags;

// Temp container that we will return the pointer to in GetCooldownTags().
// This will be a union of our CooldownTags and the Cooldown GE's cooldown tags.
UPROPERTY(Transient)
FGameplayTagContainer TempCooldownTags;
```

覆写 `UGameplayAbility::GetCooldownTags()` 来返回冷却时间标签和任何现有冷却时间 `GE` 的标签的组合；

```cpp
const FGameplayTagContainer * UPGGameplayAbility::GetCooldownTags() const
{
	FGameplayTagContainer* MutableTags = const_cast<FGameplayTagContainer*>(&TempCooldownTags);
	MutableTags->Reset(); 
	// MutableTags writes to the TempCooldownTags on the CDO so clear it in case the ability cooldown tags change (moved to a different slot)
	const FGameplayTagContainer* ParentTags = Super::GetCooldownTags();
	if (ParentTags)
	{
		MutableTags->AppendTags(*ParentTags);
	}
	MutableTags->AppendTags(CooldownTags);
	return MutableTags;
}
```

最后，再覆写`UGameplayAbility::ApplyCooldown`来注入了冷却时间标签，并将`SetByCaller`添加到冷却时间`GameplayEffectSpec`上：
```cpp
void UPGGameplayAbility::ApplyCooldown(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo) const
{
	UGameplayEffect* CooldownGE = GetCooldownGameplayEffect();
	if (CooldownGE)
	{
		FGameplayEffectSpecHandle SpecHandle = MakeOutgoingGameplayEffectSpec(CooldownGE->GetClass(), GetAbilityLevel());
		SpecHandle.Data.Get()->DynamicGrantedTags.AppendTags(CooldownTags);
		SpecHandle.Data.Get()->SetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName(  OurSetByCallerTag  )), CooldownDuration.GetValueAtLevel(GetAbilityLevel()));
		ApplyGameplayEffectSpecToOwner(Handle, ActorInfo, ActivationInfo, SpecHandle);
	}
}
```

2. 使用`MMC`；
