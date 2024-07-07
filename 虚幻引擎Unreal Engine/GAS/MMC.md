`MMC`（修饰符幅度计算，`Modifier Magnitude Calculation`），这是一个用于计算和定义`GameplayEffect`中属性修改量的类，`MMC` 允许开发者通过编程方式定义属性修改的复杂逻辑，而不仅仅是简单的数值修改；

`MMC`的功能与 [[GEEC]] 类似，但没有那么强大，最重要的是它们可以被预测。它们的唯一目的是从 `CalculateBaseMagnitude_Implementation()` 返回一个浮点值；

`MMC`可用于瞬时的、持续的、无限时间的、周期的，即所有类型的`GameplayEffect`；
# 使用场景

`MMC` 在需要复杂逻辑来决定游戏效果如何影响角色或对象时非常有用。例如，在角色受到基于其当前生命百分比的伤害时，或者在某些效果的强度需要根据游戏环境动态调整时，都可能会用到 `MMC`；
# 实现方式

`MMC` 通常通过继承 `UGameplayModMagnitudeCalculation` 类并重写 `CalculateBaseMagnitude_Implementation` 方法来实现。在这个方法中，你可以编写自定义的逻辑来决定属性修改的大小；

# 示例

以下是一个简单的示例，展示如何创建一个自定义的 `MMC` 类，该类计算一个基于其他属性值的修改量：

```cpp
#include "GameplayModMagnitudeCalculation.h"
#include "AbilitySystemComponent.h"

UCLASS()
class UMyCustomMagnitudeCalculation : public UGameplayModMagnitudeCalculation
{
    GENERATED_BODY()

public:
    virtual float CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec& Spec) const override
    {
        // 获取属性值
        float AttributeValue = 0.0f;
        if (Spec.CapturedSourceTags.GetAggregatedTags().HasTag(FGameplayTag::RequestGameplayTag(FName("SomeTag"))))
        {
            AttributeValue = 100.0; // 示例值
        }

        // 计算并返回修改量
        return AttributeValue * 2.0f;
    }
};
```

# 捕捉属性值

在`MMC`中，属性值的捕捉是通过定义`FGameplayEffectAttributeCaptureDefinition`来实现的。这个定义指定了哪个属性被捕捉、从哪里（如目标或来源）捕捉，以及是否应该捕捉属性的快照。具体步骤包括：
- **属性选择**：通过`AttributeToCapture`指定要捕捉的具体属性。
- **来源选择**：通过`AttributeSource`指定属性值的来源，可以是效果的目标（Target）或来源（Source）。
- **实时或快照**：通过`bSnapshot`决定是捕捉属性的实时值还是在效果应用时的快照值。

```cpp
// MyCustomModCalc.h
#pragma once

#include "CoreMinimal.h"
#include "GameplayEffectTypes.h"
#include "Abilities/GameplayModMagnitudeCalculation.h"
#include "MyCustomModCalc.generated.h"

UCLASS()
class MYGAME_API UMyCustomModCalc : public UGameplayModMagnitudeCalculation
{
    GENERATED_BODY()

public:
    UMyCustomModCalc();

    virtual float CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec& Spec) const override;

protected:
	//用于捕获生命值属性
    FGameplayEffectAttributeCaptureDefinition HealthDef;
};
```

```cpp
// MyCustomModCalc.cpp
#include "MyCustomModCalc.h"
#include "AbilitySystemComponent.h"
#include "GameplayEffectExtension.h"

UMyCustomModCalc::UMyCustomModCalc()
{
    //写法一
    HealthDef = FGameplayEffectAttributeCaptureDefinition(UAttributeSetHealth::GetHealthAttribute(), EGameplayEffectAttributeCaptureSource::Source, true);

	//写法二
	HealthDef.AttributeToCapture = UAttributeSetHealth::GetHealthAttribute();
	HealthDef.AttributeSource = EGameplayEffectAttributeCaptureSource::Source;
	HealthDef.bSnapshot = true;

    RelevantAttributesToCapture.Add(HealthDef);
}

float UMyCustomModCalc::CalculateBaseMagnitude_Implementation(const FGameplayEffectSpec& Spec) const
{
    // Setup evaluation parameters
    FAggregatorEvaluateParameters EvaluationParameters;

    // Capture the Health attribute value
    float Health = 0.f;
    GetCapturedAttributeMagnitude(HealthDef, Spec, EvaluationParameters, Health);

    // Example calculation: return half of the health as the magnitude
    return Health * 0.5f;
}
```
# 属性快照

快照是指在特定时间点（通常是效果应用时）捕捉并固定属性值，之后即使原始属性值改变，快照值也不会变。这在某些游戏逻辑中非常有用，比如当你想基于效果开始时的属性值来持续应用一个效果，而不希望后续的属性变化影响这个效果。

- **实时值**：如果`bSnapshot`设置为`false`，则属性值是动态的，即每次计算或查询时都会获取当前的实时值。
- **快照值**：如果`bSnapshot`设置为`true`，则在效果应用时捕捉一次值，之后即使属性本身发生变化，捕捉到的值也不会改变。
