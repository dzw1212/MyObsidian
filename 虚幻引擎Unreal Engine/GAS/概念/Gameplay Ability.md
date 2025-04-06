在`GAS`中，`GameplayAbility`是一个核心概念，代表了角色可以执行的一个具体行为或技能。这些能力可以是各种各样的，从简单的物理攻击到复杂的魔法技能，甚至是非战斗类的技能，如治疗或增益效果。

`GA`是`Actor`在游戏中可以执行的任何动作或技能，同一时间可以有多个`GA`处于活动状态；

常见的使用`GA`实现的功能：
	跳跃 `Jump`
	冲刺 `Sprint`
	射击 `Shoot`
	每个X秒被动阻挡一次攻击
	使用药水
	打开一扇门
	收集资源
	建造建筑

不应该使用`GA`实现的功能：
	基本的移动输入
	与UI界面的交互

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223152129.png)



# 流程图

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/abilityflowchartsimple.png)

带有[[Ability Tasks]]的流程：

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/abilityflowchartcomplex.png)

# 将输入绑定到ASC

要将输入绑定到`ASC`，必须首先创建一个枚举，将输入操作名称转换为字节，枚举名称必须与项目设置中输入操作的名称完全一致；

比如：
```cpp
UENUM(BlueprintType)
enum class EGDAbilityInputID : uint8
{
	// 0 None
	None			UMETA(DisplayName = "None"),
	// 1 Confirm
	Confirm			UMETA(DisplayName = "Confirm"),
	// 2 Cancel
	Cancel			UMETA(DisplayName = "Cancel"),
	// 3 LMB
	Ability1		UMETA(DisplayName = "Ability1"),
	// 4 RMB
	Ability2		UMETA(DisplayName = "Ability2"),
	// 5 Q
	Ability3		UMETA(DisplayName = "Ability3"),
	// 6 E
	Ability4		UMETA(DisplayName = "Ability4"),
	// 7 R
	Ability5		UMETA(DisplayName = "Ability5"),
	// 8 Sprint
	Sprint			UMETA(DisplayName = "Sprint"),
	// 9 Jump
	Jump			UMETA(DisplayName = "Jump")
};
```

如果`ASC`位于`Character`身上，那么建议在你的`Character`的`SetupPlayerInputComponent`中加入绑定输入到`ASC`的函数：
```cpp
void AYourCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
	Super::SetupPlayerInputComponent(PlayerInputComponent);

	//创建了一个 FTopLevelAssetPath 对象，它指定了能力输入枚举 EGDAbilityInputID 的位置。这个枚举定义了不同的能力输入ID，如确认和取消。"/Script/GASDocumentation" 是包含枚举的模块或脚本的路径
	FTopLevelAssetPath AbilityEnumAssetPath = FTopLevelAssetPath(FName("/Script/GASDocumentation"), FName("EGDAbilityInputID"));

	//BindAbilityActivationToInputComponent用于将玩家的输入绑定到能力的激活上
	AbilitySystemComponent->BindAbilityActivationToInputComponent(PlayerInputComponent, FGameplayAbilityInputBinds(
		FString("ConfirmTarget"), //触发能力的输入键
		FString("CancelTarget"),  //取消能力的输入键
		AbilityEnumAssetPath, //输入键枚举
		static_cast<int32>(EGDAbilityInputID::Confirm), 
		static_cast<int32>(EGDAbilityInputID::Cancel))
	);
}
```

如果`ASC`位于`PlayerState`上，`SetupPlayerInputComponent`内部可能存在竞争条件，因此建议在`SetupPlayerInputComponent`和`OnRep_PlayerState`两处进行输入绑定，并且使用一个布尔值进行控制（假如一处已经绑定过则另一处则跳过）；

## 按下时不立即激活

如果希望技能在输入键按下时不立即被激活，可以在`UGameplayAbility`的子类中添加一个布尔值用于控制是否在按下时激活（比如名为`bActiveOnInput`），并修改`UAbilitySystemComponent::AbilityLocalInputPressed`：

```cpp
void UGSAbilitySystemComponent::AbilityLocalInputPressed(int32 InputID)
{
	......
	UGSGameplayAbility* GA = Cast<UGSGameplayAbility>(Spec.Ability);
	if (GA && GA->bActivateOnInput)
	{
		// Ability is not active, so try to activate it
		TryActivateAbility(Spec.Handle);
	}
	......
}
```

# 赋予能力

像一个`ASC`赋予能力，`ASC`会将该能力添加到其可激活能力列表中，之后它就可以在满足`GameplayTag`的要求的情况下激活该技能；

能力应该在服务器端赋予，之后服务器会自动将`GameplayAbilitySpec`复制到拥有该能力的客户端上，其他客户端或模拟代理则不会收到该能力实例；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223153355.png)


# 激活能力

能力被激活之后，机会处于`Active`状态，直到结束或者被取消；
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223155208.png)


一般来说，一个能力会在其指定的按键按下、并且满足标签要求时被激活，但这并不是激活能力的唯一方式；`ASC`还提供了另外四种激活能力的方法：

1. 通过标签激活：
```cpp
UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilitiesByTag(const FGameplayTagContainer& GameplayTagContainer, bool bAllowRemoteActivation = true);
```

2. 通过能力类激活：
```cpp
UFUNCTION(BlueprintCallable, Category = "Abilities")
bool TryActivateAbilityByClass(TSubclassOf<UGameplayAbility> InAbilityToActivate, bool bAllowRemoteActivation = true);
```

3. 通过能力实例句柄激活：
```cpp
bool TryActivateAbility(FGameplayAbilitySpecHandle AbilityToActivate, bool bAllowRemoteActivation = true);
```

4. 通过事件激活：
```cpp
bool TriggerAbilityFromGameplayEvent(FGameplayAbilitySpecHandle AbilityToTrigger, FGameplayAbilityActorInfo* ActorInfo, FGameplayTag Tag, const FGameplayEventData* Payload, UAbilitySystemComponent& Component);
```

要通过事件激活能力，必须在`GA`中设置一个触发器，指定一个标签并为`GameplayEvent`选择一个选项，使用`UAbilitySystemBlueprintLibrary::SendGameplayEventToActor(AActor* Actor, FGameplayTag EventTag, FGameplayEventData Payload)`来发送事件；

这种方法可以传递包含用户数据的信息；

## 搭配增强输入

[[增强输入 EnhancedInput]]

建立标签与`InputAction`之间的关联：
![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223183543.png)


自定义`EnhancedInputComponent`类，并设为“默认输入组件类”，方便绑定输入事件：
![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223203847.png)

通过传递不同的`GameplayTag`来区分不同的事件；
![850](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223204336.png)


## 激活被动能力

实现自动激活并持续运行的被动能力，需要覆写`UGDGameplayAbility::OnAvatarSet`，该函数在能力被授予一个角色并且该角色的`Avatar`（如`Actor`或`Character`）被设置时调用；

建议在`UGameplayAbility`的子类中添加一个布尔值（如`bActiveAbilityOnGranted`），指定该能力是否在授予时被自动激活：
```cpp
void UGDGameplayAbility::OnAvatarSet(const FGameplayAbilityActorInfo * ActorInfo, const FGameplayAbilitySpec & Spec)
{
	Super::OnAvatarSet(ActorInfo, Spec);

	if (bActivateAbilityOnGranted)
	{
		ActorInfo->AbilitySystemComponent->TryActivateAbility(Spec.Handle, false);
	}
}
```

## 激活失败的标签

`GA`有默认的机制来提示激活失败的原因，要启用该机制，需要设置与默认失败情况相对应的标签；

将这些标签添加到项目中（如`Config/DefaultGameplayTags.ini`中）：
```cpp
+GameplayTagList=(Tag="Activation.Fail.BlockedByTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.CantAffordCost",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.IsDead",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.MissingTags",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.Networking",DevComment="")
+GameplayTagList=(Tag="Activation.Fail.OnCooldown",DevComment="")
```

然后将这些标签添加到`Config\DefaultGame.ini`中：
```cpp
[/Script/GameplayAbilities.AbilitySystemGlobals]
ActivateFailIsDeadName=Activation.Fail.IsDead
ActivateFailCooldownName=Activation.Fail.OnCooldown
ActivateFailCostName=Activation.Fail.CantAffordCost
ActivateFailTagsBlockedName=Activation.Fail.BlockedByTags
ActivateFailTagsMissingName=Activation.Fail.MissingTags
ActivateFailNetworkingName=Activation.Fail.Networking
```

然后在`UMyGameplayAbility::CanActivateAbility`中使用这些标签：
```cpp
bool UMyGameplayAbility::CanActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayTagContainer* SourceTags, const FGameplayTagContainer* TargetTags, OUT FGameplayTagContainer* OptionalRelevantTags) const
{
    if (!Super::CanActivateAbility(Handle, ActorInfo, SourceTags, TargetTags, OptionalRelevantTags))
        return false;

    // 检查是否死亡
    if (ActorInfo->IsDead())
    {
        OptionalRelevantTags->AddTag(FGameplayTag::RequestGameplayTag(FName("Activation.Fail.IsDead")));
        return false;
    }

    // 检查冷却时间
    if (IsOnCooldown(ActorInfo))
    {
        OptionalRelevantTags->AddTag(FGameplayTag::RequestGameplayTag(FName("Activation.Fail.OnCooldown")));
        return false;
    }

    // 检查能力成本
    if (!CanAffordCost(Handle, ActorInfo, SourceTags, TargetTags))
    {
        OptionalRelevantTags->AddTag(FGameplayTag::RequestGameplayTag(FName("Activation.Fail.CantAffordCost")));
        return false;
    }

    // 检查阻塞标签
    if (HasBlockingTags(ActorInfo))
    {
        OptionalRelevantTags->AddTag(FGameplayTag::RequestGameplayTag(FName("Activation.Fail.BlockedByTags")));
        return false;
    }

    // 检查缺失标签
    if (HasMissingRequiredTags(ActorInfo))
    {
        OptionalRelevantTags->AddTag(FGameplayTag::RequestGameplayTag(FName("Activation.Fail.MissingTags")));
        return false;
    }

    // 检查网络问题
    if (HasNetworkingIssues())
    {
        OptionalRelevantTags->AddTag(FGameplayTag::RequestGameplayTag(FName("Activation.Fail.Networking")));
        return false;
    }

    return true;
}
```

## 获取当前激活的能力

由于可以同时激活多个能力，所有需要遍历`ASC`的“可激活能力列表`ActivatableAbilities`”来获取当前激活的所有能力，并通过`IsActive()`筛选出其中激活的能力；

可以使用`UAbilitySystemComponent::GetActivatableAbilities`返回一个`TArray<FGameplayAbilitySpec>`返回该列表；
# 取消能力

要在`GA`内部取消能力，可以调用`CancelAbility`，这将调用`EndAbility`并将其`WasCancelled`参数设置为`true`；

要从外部取消能力，`ASC`提供了以下函数：
```cpp
/** Cancels the specified ability CDO. */
void CancelAbility(UGameplayAbility* Ability);	

/** Cancels the ability indicated by passed in spec handle. If handle is not found among reactivated abilities nothing happens. */
void CancelAbilityHandle(const FGameplayAbilitySpecHandle& AbilityHandle);

/** Cancel all abilities with the specified tags. Will not cancel the Ignore instance */
void CancelAbilities(const FGameplayTagContainer* WithTags=nullptr, const FGameplayTagContainer* WithoutTags=nullptr, UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities regardless of tags. Will not cancel the ignore instance */
void CancelAllAbilities(UGameplayAbility* Ignore=nullptr);

/** Cancels all abilities and kills any remaining instanced abilities */
virtual void DestroyActiveState();
```

# 能力等级

提升能力的等级有两种方法：
1. 先移除能力（如果已激活，则先终止该能力），再赋予提升等级后的能力；
2. 直接提升`GameplayAbilitySpec`的等级，并且标记为`dirty`，以便同步到客户端（这种方法不会终止处于激活状态的能力）；

# 能力任务

`AbilityTask` 是一种特殊的类，用于在`GA`执行期间异步执行特定的任务，允许开发者在不阻塞主游戏循环的情况下执行复杂的逻辑和时间序列；

`GAS`中自带一些能力任务：
- 使用`RootMotionSource`移动角色的任务；
- 播放动画蒙太奇的任务；
- 响应`Gameplay Attribute`变化的任务；
- 响应`Gameplay Effect`变化的任务；
- 响应玩家输入的任务；
- ETC...

BTW：`UAbilityTask` 构造函数会强制执行一个硬编码的全游戏范围最大值，即同时运行 1000 个并发能力任务，这在一些有数百个角色的游戏中需要注意；

`AbilityTask`在cpp与蓝图均可以执行：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223153054.png)


## 使用能力任务

比如播放动画蒙太奇的任务：
```cpp
UGDAT_PlayMontageAndWaitForEvent* Task = UGDAT_PlayMontageAndWaitForEvent::PlayMontageAndWaitForEvent(this, NAME_None, MontageToPlay, FGameplayTagContainer(), 1.0f, NAME_None, false, 1.0f);
Task->OnBlendOut.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnCompleted.AddDynamic(this, &UGDGA_FireGun::OnCompleted);
Task->OnInterrupted.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->OnCancelled.AddDynamic(this, &UGDGA_FireGun::OnCancelled);
Task->EventReceived.AddDynamic(this, &UGDGA_FireGun::EventReceived);
Task->ReadyForActivation();
```

# 能力类的属性

## 标签

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223164911.png)


![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223164823.png)

## 实例化策略

`Non-Instanced`拥有最佳性能，就类似于一组静态函数，但是由于其没有任何示例，也就不能储存任何信息；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223165917.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223165521.png)


## 网络执行策略

`Local Only`适合一些纯客户端表现、且不需要与其他玩家交互的技能；
`Server Only`适合被动技能、光环等，
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223170007.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223165927.png)

**如果`Ability`在客户端激活，并且策略是`Local Predicted`，客户端会立即执行（没有延迟），然后再在服务器端验证后执行（同步给其他玩家），注意客户端不能在激活任务后立即`EndAbility`，这样会导致能力无法同步到服务器端**；


# 示例

假设你想创建一个简单的火球技能，你可能会创建一个继承自`UGameplayAbility`的类，并重写`ActivateAbility`方法来定义释放火球的逻辑。

```cpp
#include "Abilities/GameplayAbility.h"
#include "GameplayEffect.h"

class UFireballAbility : public UGameplayAbility
{
    GENERATED_BODY()

public:
    // 构造函数中配置能力属性
    UFireballAbility();

    // 重写激活能力的方法
    virtual void ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData) override;
};

void UFireballAbility::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
    // 在这里实现释放火球的逻辑，比如生成火球实体，设置其飞行方向和速度等
}
```

在实际项目中，你还需要在游戏的其他部分（如角色类或游戏模式类）中配置和使用这个能力，包括将其添加到角色的`AbilitySystemComponent`中。

# Cost与Cooldown

`GameplayAbility`基类中，自带`Cost`与`Cooldown`的机制；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240716074352.png)

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240716074420.png)

当`CommitAbility`时，会去检测`Cost`与`Cooldown`是否满足条件；也可以通过`CommitAbilityCost`或`CommitAbilityCooldown`单独检测某一项；

```cpp
//UGameplayAbility::CommitCheck

if (!AbilitySystemGlobals.ShouldIgnoreCooldowns() && !CheckCooldown(Handle, ActorInfo, OptionalRelevantTags))
{
	return false;
}

if (!AbilitySystemGlobals.ShouldIgnoreCosts() && !CheckCost(Handle, ActorInfo, OptionalRelevantTags))
{
	return false;
}
```

`CheckCost`中检测能否对属性值执行对应的操作，如果未通过，则会加上`ActivateFailCostTag`标记；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240716074945.png)

`CheckCooldown`中则是检测是否存在对应的冷却标签，如果存在则说明冷却未完成，添加`ActivateFailCooldownTag`标记；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240716075240.png)


# 能力Actor

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240715225934.png)

## TargetActor

`GameplayAbilityTargetActor` 是一个用于处理目标选择的类。它提供了一种机制，让玩家或AI能够选择目标或目标区域，以便应用能力效果。`GameplayAbilityTargetActor` 可以用于多种目标选择方式，例如单个目标、区域目标、射线检测等。

`GameplayAbilityTargetActor`的子类一般都会重写`StartTargeting(UGameplayAbility* Ability)`和`ConfirmTargetingAndContinue()`方法：

- `StartTargeting`：用于初始化目标选择过程，它通常会设置目标选择器的初始状态，比如显示目标指示器、设置目标选择的范围和规则等；当一个`Gameplay Ability`开始目标选择过程时，会调用此方法，通常在玩家激活一个需要选择目标的能力时触发；
- `ConfirmTargetingAndContinue`：用于确认目标选择并继续执行能力，它会处理目标选择的结果，并将这些结果传递给能力系统，以便后续的能力执行；当目标选择过程完成并且玩家确认了选择的目标时，会调用次方法，通常在玩家点击确认按钮或完成目标选择操作时触发；

虚幻中预先提供了一些`Target Actor`的子类：

### Radius

`AGameplayAbilityTargetActor_Radius` 会定义一个圆形区域，通常通过一个半径值来确定该区域的大小；

