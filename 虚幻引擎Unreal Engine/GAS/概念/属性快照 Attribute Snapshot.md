在GAS中，属性快照（`Snapshot`）是指在某个特定时间点（一般是创建`GESpec`时）捕获属性的值，并在之后的计算中使用这个捕获的值，而不是实时获取当前值。

快照的主要作用是确保在执行`Gameplay Effect`时，使用的是一致的属性值，避免因为属性值的变化导致计算结果不一致；

1. **一致性**: 确保在整个效果执行过程中使用的是同一个属性值，避免因为属性值的变化导致计算结果不一致。

2. **性能优化**: 减少在效果执行过程中多次获取属性值的开销。

比如说，`Demage`属性适合被快照，一般来说技能一旦创建后其伤害就是固定的，不应该吃到后续的加成；

# GE Spec

一般快照会配合`GESpec`使用，在`GESpec`创建时对属性值进行快照：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240722025017.png)


```cpp
// 获取效果规格
const FGameplayEffectSpec& Spec = ExecutionParams.GetOwningSpec();

// 获取标签
const FGameplayTagContainer* SourceTags = Spec.CapturedSourceTags.GetAggregatedTags();
const FGameplayTagContainer* TargetTags = Spec.CapturedTargetTags.GetAggregatedTags();

FAggregatorEvaluateParameters EvaluationParameters;
EvaluationParameters.SourceTags = SourceTags;
EvaluationParameters.TargetTags = TargetTags;
```

此外，还需要通过`GESpec`获取`SetByCaller`的数值；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240722025343.png)


```cpp
float Damage = 0.0f;
// Capture optional damage value set on the damage GE as a CalculationModifier under the ExecutionCalculation
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().DamageDef, EvaluationParameters, Damage);
// Add SetByCaller damage if it exists
Damage += FMath::Max<float>(Spec.GetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName("Data.Damage")), false, -1.0f), 0.0f);
```

# 相关宏

## 声明属性捕获

```cpp
#define DECLARE_ATTRIBUTE_CAPTUREDEF(P) \
	FProperty* P##Property; \
	FGameplayEffectAttributeCaptureDefinition P##Def; \
```

示例：
```cpp
DECLARE_ATTRIBUTE_CAPTUREDEF(Armor);
```
## 定义属性捕获

```cpp
#define DEFINE_ATTRIBUTE_CAPTUREDEF(S, P, T, B) \
{ \
	P##Property = FindFieldChecked<FProperty>(S::StaticClass(), GET_MEMBER_NAME_CHECKED(S, P)); \
	P##Def = FGameplayEffectAttributeCaptureDefinition(P##Property, EGameplayEffectAttributeCaptureSource::T, B); \
}
```

示例：
```cpp
//参数1：AttributeSet类
//参数2：属性名称
//参数3：指定要从源（Source）还是目标（Target）捕获属性
//参数4：是否快照
DEFINE_ATTRIBUTE_CAPTUREDEF(UGDAttributeSetBase, Armor, Target, false);
```

## 示例代码

```cpp
struct GDDamageStatics
{
	DECLARE_ATTRIBUTE_CAPTUREDEF(Armor);
	DECLARE_ATTRIBUTE_CAPTUREDEF(Damage);

	GDDamageStatics()
	{
		DEFINE_ATTRIBUTE_CAPTUREDEF(UGDAttributeSetBase, Damage, Source, true);
		DEFINE_ATTRIBUTE_CAPTUREDEF(UGDAttributeSetBase, Armor, Target, false);
	}
};

//单例结构体，确保只执行一次
static const GDDamageStatics& DamageStatics()
{
	static GDDamageStatics DStatics;
	return DStatics;
}
```

```cpp
UGDDamageExecCalculation::UGDDamageExecCalculation()
{
	RelevantAttributesToCapture.Add(DamageStatics().DamageDef);
	RelevantAttributesToCapture.Add(DamageStatics().ArmorDef);
}
```

```cpp
float Armor = 0.0f;
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().ArmorDef, EvaluationParameters, Armor);
Armor = FMath::Max<float>(Armor, 0.0f);

float Damage = 0.0f;
ExecutionParams.AttemptCalculateCapturedAttributeMagnitude(DamageStatics().DamageDef, EvaluationParameters, Damage);
Damage += FMath::Max<float>(Spec.GetSetByCallerMagnitude(FGameplayTag::RequestGameplayTag(FName("Data.Damage")), false, -1.0f), 0.0f);

float UnmitigatedDamage = Damage; // Can multiply any damage boosters here

float MitigatedDamage = (UnmitigatedDamage) * (100 / (100 + Armor));

if (MitigatedDamage > 0.f)
	OutExecutionOutput.AddOutputModifier(FGameplayModifierEvaluatedData(DamageStatics().DamageProperty, EGameplayModOp::Additive, MitigatedDamage));
	
```

