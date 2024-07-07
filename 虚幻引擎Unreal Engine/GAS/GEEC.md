`GEEC`（游戏效果执行计算，`Gameplay Effect Execution Calculation`），是`GameplayEffect`对`ASC`进行更改的最强大的方法；类似于[[MMC]]，它也可以捕获属性并对其进行快照，但不同的是`GEEC`能够更改多个属性，并且更加灵活，与此同时牺牲的是可预测性；

`GEEC`仅可用于瞬时和周期性的`GameplayEffect`；

```cpp
// MyDamageExecutionCalc.h
#pragma once

#include "CoreMinimal.h"
#include "GameplayEffectExecutionCalculation.h"
#include "MyDamageExecutionCalc.generated.h"

UCLASS()
class MYGAME_API UMyDamageExecutionCalc : public UGameplayEffectExecutionCalculation
{
    GENERATED_BODY()

public:
    UMyDamageExecutionCalc();

    virtual void Execute_Implementation(const FGameplayEffectCustomExecutionParameters& ExecutionParams, FGameplayEffectCustomExecutionOutput& OutExecutionOutput) const override;
};
```

```cpp
// MyDamageExecutionCalc.cpp
#include "MyDamageExecutionCalc.h"
#include "AbilitySystemComponent.h"
#include "GameplayEffectTypes.h"

UMyDamageExecutionCalc::UMyDamageExecutionCalc()
{
    // 在这里可以添加相关的属性依赖，例如依赖于攻击力或防御力
    RelevantAttributesToCapture.Add(FAggregatorEvaluateMetaData(TEXT("攻击力"), EGameplayEffectScopedModifierAggregatorType::CapturedAttributeBacked));
}

void UMyDamageExecutionCalc::Execute_Implementation(const FGameplayEffectCustomExecutionParameters& ExecutionParams, FGameplayEffectCustomExecutionOutput& OutExecutionOutput) const
{
    float AttackPower = 0.f;
    ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(FAggregatorEvaluateMetaData(TEXT("攻击力"), EGameplayEffectScopedModifierAggregatorType::CapturedAttributeBacked), FAggregatorEvaluateParameters(), AttackPower);

    float Damage = AttackPower * 2.0f; // 假设伤害是攻击力的两倍

    OutExecutionOutput.AddOutputModifier(FGameplayModifierEvaluatedData(FAggregatorEvaluateMetaData(TEXT("生命值"), EGameplayEffectScopedModifierAggregatorType::OutgoingGE), EGameplayModOp::Additive, -Damage));
}
```