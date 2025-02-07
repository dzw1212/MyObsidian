在`GAS`中，`Gameplay Attributes`是一个由结构体`FGameplayAttributeData`定义的浮点数值，用于表示、存储和管理游戏中角色或对象的各种属性（如生命值、法力值、速度等）。这些属性是游戏逻辑和游戏玩法中不可或缺的部分，它们可以被修改和查询（一般建议只由`Gameplay Effect`进行修改，以方便`ASC`预测变化），以支持复杂的游戏能力、效果和交互。

*提示：如果你不希望某个属性出现在编辑器的属性列表中，可以使用 Meta = (HideInDetailsView) 属性指定符*

# 核心特点

- **数据驱动**：`GameplayAttribute`通常与数据表（如`UDataTable`）结合使用，允许游戏设计师通过配置而非硬编码的方式定义和修改属性。
- **高度可扩展**：开发者可以根据需要自定义和扩展属性，以支持游戏的特定需求。
- **与GameplayEffect交互**：在`GAS`中，`GameplayEffect`用于修改`GameplayAttribute`，例如，一个“治疗”效果可能会增加角色的生命值属性。
- **网络复制**：`GAS`设计用于多人游戏，因此`GameplayAttribute`支持网络复制，确保所有玩家客户端的游戏状态同步。

# 基础值与当前值

一个属性由两个值组成——`BaseValue`（基础值）和 `CurrentValue`（当前值）。`BaseValue` 是属性的永久值，而 `CurrentValue` 是 `BaseValue` 加上来自 `GameplayEffects` 的临时修改。

例如，你的角色可能有一个移动速度属性，`BaseValue` 为 600 单位/秒。由于还没有 `GameplayEffects` 修改移动速度，`CurrentValue` 也是 600 单位/秒。如果她获得一个临时的 50 单位/秒的移动速度增益，`BaseValue` 保持不变为 600 单位/秒，而 `CurrentValue` 现在是 600 + 50，总共 650 单位/秒。当移动速度增益消失时，`CurrentValue` 恢复到 `BaseValue` 的 600 单位/秒。

## GameplayEffect与属性值的修改

[[Gameplay Effect#分类]]

- 瞬时性 *Instant* 和周期性 *Duration* 的`Gameplay Effect`一般修改基础值；
- 持续性 *Infinite* 的`Gameplay Effect`一般修改当前值；

# 使用属性

## 定义属性

属性只能在C++中、对应属性集的头文件中定义；

建议在每个属性集的头文件顶部添加以下的宏：
```cpp
// Uses macros from AttributeSet.h
#define ATTRIBUTE_ACCESSORS(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_GETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_SETTER(PropertyName) \
	GAMEPLAYATTRIBUTE_VALUE_INITTER(PropertyName)
```

比如定义一个生命值属性：
```cpp
UPROPERTY(BlueprintReadOnly, Category = "Health", ReplicatedUsing = OnRep_Health)
FGameplayAttributeData Health;
ATTRIBUTE_ACCESSORS(UGDAttributeSetBase, Health)
```

当`Health`的值在服务器上改变并复制到客户端时，`OnRep_Health`函数会被调用：
```cpp
UFUNCTION()
virtual void OnRep_Health(const FGameplayAttributeData& OldHealth);
```

`OnRep_<AttributeName>`中通常使用`GAMEPLAYATTRIBUTE_REPNOTIFY`宏来调用`GAS`中的标准属性复制通知逻辑：
```cpp
void UGDAttributeSetBase::OnRep_Health(const FGameplayAttributeData& OldHealth)
{
	GAMEPLAYATTRIBUTE_REPNOTIFY(UGDAttributeSetBase, Health, OldHealth);
	//其他处理
}
```

对于需要在网络环境中同步的属性，通常应该将它们添加到 `GetLifetimeReplicatedProps`中，这是确保多人游戏中数据一致性和正确性的关键步骤；
```cpp
void UGDAttributeSetBase::GetLifetimeReplicatedProps(TArray<FLifetimeProperty>& OutLifetimeProps) const
{
	Super::GetLifetimeReplicatedProps(OutLifetimeProps);

	DOREPLIFETIME_CONDITION_NOTIFY(UGDAttributeSetBase, Health, COND_None, REPNOTIFY_Always);
}
```

如果是元属性这样不需要复制的，则无需`OnRep`与`GetLifetimeReplicatedProps`；
### Getter与Setter宏

使用宏来定义`GameplayAttribute`的`getter`和`setter`方法是一种简化代码和提高可维护性的技术。这些宏背后的原理是预处理器指令，它们在编译时会展开成实际的代码，从而减少了手动编写重复代码的需要。

假设我们有一个宏定义如下：

```cpp
#define GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
    FORCEINLINE const FGameplayAttributeData* Get##PropertyName##() const \
    { \
        return &PropertyName; \
    }
```

当你在`UAttributeSet`派生类中使用这个宏时：

```cpp
GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UYourAttributeSet, Health)
```

预处理器会将它展开为：

```cpp
FORCEINLINE const FGameplayAttributeData* GetHealth() const
{
    return &Health;
}
```

- **减少重复代码**：对于每个属性的getter和setter，如果手动编写，代码会非常相似，只是属性名不同。使用宏可以避免这种重复，使代码更加简洁。
- **易于维护**：如果getter和setter的通用模式需要更改，你只需修改宏的定义，而不是每个地方的实现。这使得维护和更新变得更加容易。
- **提高一致性**：宏确保了所有属性的getter和setter遵循相同的模式，减少了因手动编码错误而引入的不一致性。

---

- **GAMEPLAYATTRIBUTE_VALUE_GETTER**

`GAMEPLAYATTRIBUTE_VALUE_GETTER`宏用于创建一个函数，该函数返回属性的当前值。这个宏使得访问属性值变得简单直接，无需直接与`FGameplayAttributeData`结构交互。

```cpp
GAMEPLAYATTRIBUTE_VALUE_GETTER(AttributeName)
```

这将生成一个名为`GetAttributeName()`的函数，用于返回`AttributeName`属性的当前值。

 - **GAMEPLAYATTRIBUTE_VALUE_SETTER**

`GAMEPLAYATTRIBUTE_VALUE_SETTER`宏用于创建一个函数，该函数允许你设置属性的当前值。这是修改属性值的标准方法，确保所有相关的游戏逻辑和网络复制都能正确处理属性值的变化。

```cpp
GAMEPLAYATTRIBUTE_VALUE_SETTER(AttributeName)
```

这将生成一个名为`SetAttributeName(float NewValue)`的函数，用于设置`AttributeName`属性的当前值。

- **GAMEPLAYATTRIBUTE_VALUE_INITTER**

`GAMEPLAYATTRIBUTE_VALUE_INITTER`宏用于创建一个函数，该函数用于初始化属性的值。这通常在属性集（Attribute Set）被创建时使用，以确保所有属性都有一个明确的初始值。

```cpp
GAMEPLAYATTRIBUTE_VALUE_INITTER(AttributeName)
```

这将生成一个名为`InitAttributeName(float InitialValue)`的函数，用于初始化`AttributeName`属性的值。

## 初始化属性

初始化一个属性有多种方法，`UE`推荐使用一个瞬时的`GameplayEffect`来完成该操作；

```cpp
void AMyCharacter::InitializeAttributes()
{
    if (AbilitySystemComponent)
    {
        // Load the GameplayEffect class
        static ConstructorHelpers::FObjectFinder<UGameplayEffect> GE_InitializeAttributes(TEXT("/Game/Path/To/GE_InitializeAttributes.GE_InitializeAttributes"));
        
        if (GE_InitializeAttributes.Succeeded())
        {
            FGameplayEffectContextHandle EffectContext = AbilitySystemComponent->MakeEffectContext();
            EffectContext.AddSourceObject(this);

            FGameplayEffectSpecHandle NewHandle = AbilitySystemComponent->MakeOutgoingSpec(GE_InitializeAttributes.Object, 1, EffectContext);
            if (NewHandle.IsValid())
            {
                AbilitySystemComponent->ApplyGameplayEffectSpecToSelf(*NewHandle.Data.Get());
            }
        }
    }
}
```
## 属性修改预处理

可以使用`PreAttributeChange`对属性的修改进行预处理，其传入`NewValue`的引用，一般会在该函数中对值进行`clamp`处理；

在此之后，属性系统会根据游戏逻辑（如游戏效果的应用）来修改属性值；这可能涉及到多种计算，包括但不限于加成、乘法效果等；

```cpp
void UYourAbilitySystemComponent::PreAttributeChange(const FGameplayAttribute& Attribute, float& NewValue)
{
	Super::PreAttributeChange(Attribute, NewValue);

    // 检查是否是我们关心的属性，例如“Health”
    if (Attribute == GetHealthAttribute()) //由宏创建
    {
        // 将新的健康值限制在一个特定的范围内，比如100到1000
        NewValue = FMath::Clamp<float>(NewValue, 100.0f, 1000.0f);
    }
}
```

需要注意的是，只有从`ASC`中访问该属性才会应用`PreAttributeChange`，如果是通过其他途径访问属性值（比如`GameplayEffectExecutionCalculations`和`ModifierMagnitudeCalculations`）需要重新进行`clamp`处理；

推荐`PreAttributeChange`中仅进行限值处理，和游戏逻辑相关的操作应该放在`UAbilitySystemComponent::GetGameplayAttributeValueChangeDelegate`中进行处理；

## 属性修改后处理

可以使用`PostGameplayEffectExecute`对属性的修改进行后处理；

在属性值被修改并且游戏效果被执行后，`PostGameplayEffectExecute`被调用；这个函数是一个后处理步骤，允许开发者在游戏效果完全应用后执行额外的逻辑；

例如，可以在这里检查属性值是否触发了某些特定的游戏逻辑，如角色死亡、状态变化等；

```cpp
void UYourAbilitySystemComponent::PostGameplayEffectExecute(const FGameplayEffectModCallbackData& Data)
{
    Super::PostGameplayEffectExecute(Data);

    // 获取影响的属性
    const FGameplayAttribute& Attribute = Data.EvaluatedData.Attribute;

    // 检查是否是我们关心的属性，例如“Health”
    if (Attribute == HealthAttribute())
    {
        // 获取当前的健康值
        float CurrentHealth = GetAttributeValue(HealthAttribute());

        // 检查健康值是否低于某个阈值，例如0
        if (CurrentHealth <= 0)
        {
            // 触发角色死亡逻辑
            HandleDeath();
        }
    }
}
```

## 属性聚合器创建时

[[Attribute Aggregator]]

该回调为用户提供了一个机会来自定义或配置新创建的 `AttributeAggregator`，比如用户希望筛选所有正面的属性修改器，或者说只应用数值最大的负面的属性修改器，这都是很常见的需求；

```cpp
virtual void OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const override;
```

```cpp
void UGSAttributeSetBase::OnAttributeAggregatorCreated(const FGameplayAttribute& Attribute, FAggregator* NewAggregator) const
{
	Super::OnAttributeAggregatorCreated(Attribute, NewAggregator);

	if (!NewAggregator)
		return;
		
	//计算最终属性值时，将只考虑最负面（即最大负值）的修改器，而所有正面的修改器都将被累加
	if (Attribute == GetMoveSpeedAttribute())
	{
		NewAggregator->EvaluationMetaData = &FAggregatorEvaluateMetaDataLibrary::MostNegativeMod_AllPositiveMods;
	}
}
```
## 监听属性修改

```cpp
AbilitySystemComponent->GetGameplayAttributeValueChangeDelegate(AttributeSetBase->GetHealthAttribute()).AddUObject(this, &AGDPlayerState::HealthChanged);
```

回调中会提供一个名为`FOnAttributeChangeData`的参数，其中包含`NewValue`、`OldValue`和`FGameplayEffectModCallbackData`（该回调数据仅用于服务器端）；

```cpp
virtual void HealthChanged(const FOnAttributeChangeData& Data);
```


# 特殊属性类型
## 元属性

元属性（`Meta Attribute`）是一种特殊的属性，它们用于影响或修改其他属性的行为，而不直接表示一个具体的、可观察的属性；

假设有一个属性叫做`Health`（生命值），还有一个元属性叫做`HealthRegenerationRate`（生命恢复速率）。在这里，`Health`是一个普通属性，直接影响角色的生命值状态；而`HealthRegenerationRate`是一个元属性，它用来确定`Health`每秒恢复的量，但它自身并不直接表现为角色的具体生命值；

元属性为诸如伤害和治疗之类的事情提供了良好的逻辑分离，区分了“我们造成了多少伤害？”和“我们如何处理这些伤害？”。这种逻辑分离意味着我们的 `Gameplay Effects` 和执行计算不需要知道目标如何处理伤害。`Gameplay Effect` 确定伤害量，然后 `AttributeSet` 决定如何处理该伤害。并不是所有角色都可能具有相同的属性，特别是如果你使用子类化的 `AttributeSets`。基础 `AttributeSet` 类可能只有一个生命值属性，但子类化的 `AttributeSet` 可能会添加一个护盾属性。具有护盾属性的子类化 `AttributeSet` 会与基础 `AttributeSet` 类不同地分配所受的伤害。

元属性通常不会被复制，其没有持久性，会被每个`Gameplay Effect`所覆盖；

元属性并不是必要的，但它有如下几个优点：
1. 更方便区分“主要属性”和“辅助属性”，便于理解；
2. 帮助简化属性之间的依赖关系；
3. 更偏向于数据驱动，在不修改代码的情况下调整平衡；

### 使用实例

比如生命值为普通属性，伤害值为元属性，用于辅助伤害计算；

1. 创建属性
```cpp
// MyAttributeSet.h
#pragma once

#include "CoreMinimal.h"
#include "AttributeSet.h"
#include "AbilitySystemComponent.h"
#include "MyAttributeSet.generated.h"

UCLASS()
class UMyAttributeSet : public UAttributeSet
{
    GENERATED_BODY()

public:
    UPROPERTY(BlueprintReadOnly, Category = "Attributes")
    FGameplayAttributeData Health;

    // Meta attribute for temporary damage storage
    UPROPERTY(BlueprintReadOnly, Category = "Attributes")
    FGameplayAttributeData Damage;
};
```

2. 创建一个`GameplayEffect`，使用元属性`Damage`存储伤害值
```cpp
// Create a new GameplayEffect object
UGameplayEffect* DamageEffect = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("ApplyDamage")));
DamageEffect->DurationPolicy = EGameplayEffectDurationType::Instant;

// Modifier to store damage in the Damage meta attribute
FGameplayModifierInfo DamageStorageModifier;
DamageStorageModifier.Attribute = UMyAttributeSet::GetDamageAttribute();
DamageStorageModifier.ModifierOp = EGameplayModOp::Additive;
DamageStorageModifier.ModifierMagnitude = FScalableFloat(100.f); // Assume 100 damage for example

DamageEffect->Modifiers.Add(DamageStorageModifier);

// Modifier to apply the stored damage to Health
FGameplayModifierInfo HealthModifier;
HealthModifier.Attribute = UMyAttributeSet::GetHealthAttribute();
HealthModifier.ModifierOp = EGameplayModOp::Additive;
HealthModifier.ModifierMagnitude = FAttributeBasedFloat();
HealthModifier.ModifierMagnitude.Coefficient = -1.f;
HealthModifier.ModifierMagnitude.PreMultiplyAdditiveValue = FGameplayEffectAttributeCaptureDefinition(UMyAttributeSet::GetDamageAttribute(), EGameplayEffectAttributeCaptureSource::Source, true);

DamageEffect->Modifiers.Add(HealthModifier);
```

3. 应用`GameplayEffect`
```cpp
UAbilitySystemComponent* ASC = ...; // 获取目标的AbilitySystemComponent
ASC->ApplyGameplayEffectToSelf(DamageEffect, 1.0f, ASC->MakeEffectContext());
```
## 派生属性

派生属性（`Derived Attribute`）是一种特殊的`GameplayAttribute`，它们的值是基于一个或多个其他属性（基础属性）计算得出的。这与普通的属性不同，后者的值通常直接设定或通过游戏事件直接修改；

可以使用*Infinite*类型的`GameplayEffect`，和[[修饰符幅度计算 MMC]]来创建派生属性；

比如说衍生属性总防御力=基础防御+装备防御加成+Buff防御加成，公式中的任意属性的修改都会触发衍生属性的重新计算；

*注意：如果在PIE（Play In Editor）中使用多个客户端，需要在编辑器首选项中禁用“Run Under One Process”，否则派生属性在第一个客户端之外的其他客户端上，其独立属性更新时不会更新；*

### 使用示例

假如我需要一个衍生属性D，其值为普通属性$A+B*C$：

1. 创建属性
```cpp
UCLASS()
class UMyAttributeSet : public UAttributeSet
{
    GENERATED_BODY()

public:
    UPROPERTY(BlueprintReadOnly, Category = "Attributes")
    FGameplayAttributeData A;

    UPROPERTY(BlueprintReadOnly, Category = "Attributes")
    FGameplayAttributeData B;

    UPROPERTY(BlueprintReadOnly, Category = "Attributes")
    FGameplayAttributeData C;

    UPROPERTY(BlueprintReadOnly, Category = "Attributes")
    FGameplayAttributeData D;
};
```

2. 创建`MMC`实现计算逻辑
```cpp
#include "GameplayModMagnitudeCalculation.h"
#include "AbilitySystemComponent.h"

UCLASS()
class UMyMagnitudeCalculation : public UGameplayModMagnitudeCalculation
{
    GENERATED_BODY()

public:
    virtual float CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec& Spec) const override
    {
        const FGameplayEffectSpecHandle SpecHandle = FGameplayEffectSpecHandle(&Spec);
        float ValueA = 0.0f;
        float ValueB = 0.0f;
        float ValueC = 0.0f;

        if (Spec.GetSetByCallerMagnitude(FGameplayAttributeData(A).GetGameplayAttribute(), SpecHandle, ValueA) &&
            Spec.GetSetByCallerMagnitude(FGameplayAttributeData(B).GetGameplayAttribute(), SpecHandle, ValueB) &&
            Spec.GetSetByCallerMagnitude(FGameplayAttributeData(C).GetGameplayAttribute(), SpecHandle, ValueC))
        {
            return ValueA + (ValueB * ValueC);
        }

        return 0.0f;
    }
};
```

3. 创建一个`GameplayEffect`来应用该计算逻辑
```cpp
UGameplayEffect* Effect = NewObject<UGameplayEffect>(GetTransientPackage(), FName(TEXT("CalculateD")));
Effect->DurationPolicy = EGameplayEffectDurationType::Infinite;

FGameplayModifierInfo ModifierInfo;
ModifierInfo.Attribute = UMyAttributeSet::GetDAttribute();
ModifierInfo.ModifierOp = EGameplayModOp::Override;
ModifierInfo.ModifierMagnitude = FGameplayEffectModifierMagnitude(UMyMagnitudeCalculation::StaticClass());

Effect->Modifiers.Add(ModifierInfo);
```

# 属性集

属性集`AttributeSet`负责定义、保存、管理对属性值的修改，用户应该子类化`UAttributeSet`，在`OwnerActor`的构造函数中创建一个属性集或自动将其注册到`ASC`中（以上操作需在C++中完成）；

属性集的内存开销可以忽略不记，可以让所以角色共享一个大型的属性集（只使用其中需要的属性，忽略未使用的属性），也可以创建多个属性集来进行属性分组；

虽然`ASC`身上的属性集可以有多个，但是同类型的属性集最多只能有一个，否则获取时会产生歧义：
```cpp
UAttributeSet* MyAttributeSet = MyAbilitySystemComponent->GetAttributeSet<UAttributeSet>();
```

## 属性集设计

一个`ASC`可以拥有一个或多个属性集。属性集的内存开销可以忽略不计，因此使用多少个属性集取决于用户的设计；

可以在游戏中为每个`Actor`共享一个大型的单一属性集，并根据需要使用属性，同时忽略未使用的属性；也可以选择拥有多个属性集，代表属性的分组，并根据需要选择性地添加到您的`Actors`中。例如，您可以有一个用于健康相关属性的属性集，一个用于法力相关属性的属性集，等等。在MOBA游戏中，英雄可能需要法力，但小兵可能不需要。因此，英雄会获得法力属性集，而小兵则不会；

**虽然可以拥有多个属性集，但不应在一个`ASC`上拥有多个相同类的属性集**。如果有多个来自相同类的属性集，系统将不知道使用哪个属性集，并且只会选择一个；
## 运行时添加和移除

```cpp
//添加属性集
AbilitySystemComponent->GetSpawnedAttributes_Mutable().AddUnique(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();

//移除属性集
AbilitySystemComponent->GetSpawnedAttributes_Mutable().Remove(WeaponAttributeSetPointer);
AbilitySystemComponent->ForceReplication();
```

运行时移除属性集可能会导致危险 —— 比如客户端先于服务器端移除了某个属性集，而服务器端又复制了一些属性到客户端，此时客户端使用这些属性会找不到对应的属性集，从而导致宕机；

# 物品属性

有几种方法可以实现带有属性（武器弹药、护甲耐久度等）的可装备物品。所有这些方法都是直接在物品上存储数值。这对于在其生命周期内可被多个玩家装备的物品来说是必要的。

与其使用属性，不如在物品类实例上存储普通的浮点值。`Fortnite`和`GASShooter`就是这样处理枪支弹药的。对于一把枪，直接在枪实例上存储最大弹夹容量、当前弹夹中的弹药、备用弹药等为可复制的浮点数（`COND_OwnerOnly`）。如果武器共享备用弹药，您可以将备用弹药作为属性移动到角色上，并放在一个共享弹药的属性集中（重新装填能力可以使用一个`Cost GE`从备用弹药中提取到枪的浮点弹夹弹药中）。由于您没有使用属性来表示当前弹夹弹药，您需要重写`UGameplayAbility`中的一些函数，以检查和应用对枪上浮点数的消耗。将枪作为`GameplayAbilitySpec`中的`SourceObject`，在授予能力时意味着您可以在能力内部访问授予能力的枪。

为了防止枪在自动射击时复制回弹药量并覆盖本地弹药量，在`PreReplication()`中禁用复制，当玩家有`IsFiring`的`GameplayTag`时：
```cpp
void AGSWeapon::PreReplication(IRepChangedPropertyTracker& ChangedPropertyTracker)
{
    Super::PreReplication(ChangedPropertyTracker);

    DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, PrimaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
    
    DOREPLIFETIME_ACTIVE_OVERRIDE(AGSWeapon, SecondaryClipAmmo, (IsValid(AbilitySystemComponent) && !AbilitySystemComponent->HasMatchingGameplayTag(WeaponIsFiringTag)));
}
```




# 实现方式

`GameplayAttribute`通过C++类`FGameplayAttribute`实现。开发者需要在自己的代码中定义具体的属性，通常是通过继承`UAttributeSet`类来实现的。`UAttributeSet`类为属性提供了存储、管理和复制的功能。

# 示例

假设你正在开发一个有生命值（`Health`）和法力值（`Mana`）的角色，你可能会这样定义你的`AttributeSet`：

```cpp
#include "AbilitySystemComponent.h"
#include "AttributeSet.h"
#include "GameplayTagContainer.h"
#include "YourAttributeSet.generated.h"

UCLASS()
class YOURGAME_API UYourAttributeSet : public UAttributeSet
{
    GENERATED_BODY()

public:
    UYourAttributeSet();

    // 使用GAMEPLAYATTRIBUTE_PROPERTY_GETTER宏来声明属性的Getter函数
    // 生命值
    UPROPERTY(BlueprintReadOnly, Category = "Attributes|Health")
    FGameplayAttributeData Health;
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UYourAttributeSet, Health)

    // 法力值
    UPROPERTY(BlueprintReadOnly, Category = "Attributes|Mana")
    FGameplayAttributeData Mana;
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UYourAttributeSet, Mana)

    // 你可以继续添加更多属性...
};
```

在这个例子中，`Health`和`Mana`属性被定义为`FGameplayAttributeData`类型，这是`GAS`用于存储属性值的标准方式。通过在`UAttributeSet`派生类中定义这些属性，你可以在游戏中使用`GAS`的功能来修改和查询这些值。

