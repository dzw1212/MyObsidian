`GameplayAbilities` 只能在一帧内执行，如果要执行随着时间推移而发生的动作，或者需要对稍后某个时间点的委托做出响应，就需要使用称为 `AbilityTasks` 的功能；

`AbilityTask` 是`GAS`的一部分，用于处理异步任务和事件。`AbilityTask` 允许你在[[Gameplay Ability]]执行过程中等待特定事件或条件的发生，并在这些事件或条件满足时执行相应的逻辑；

# 常见Task

一些常用的`Task`：

1. `UAbilityTask_WaitDelay`：等待指定的时间；

```cpp
//等待2s后执行
 UAbilityTask_WaitDelay* WaitTask = UAbilityTask_WaitDelay::WaitDelay(this, 2.0f);
 WaitTask->OnFinish.AddDynamic(this, &UMyGameplayAbility::OnWaitDelayFinish);
```

2. `UAbilityTask_WaitGameplayEvent`：等待特定的游戏事件（`Gameplay Event`）；

```cpp
//EventTag 是指带有特定标签（Tag）的所有游戏事件（Gameplay Event）
 UAbilityTask_WaitGameplayEvent* EventTask = UAbilityTask_WaitGameplayEvent::WaitGameplayEvent(this, EventTag);
 EventTask->EventReceived.AddDynamic(this, &UMyGameplayAbility::OnGameplayEventReceived);
```

3. `UAbilityTask_WaitTargetData`：等待目标数据（`Target Data`）的返回；

```cpp
//TargetingConfirmation::Instant：表示目标数据会立即确认，无需玩家进一步操作
//TargetActor：是一个目标选择器（Target Actor），例如 AGameplayAbilityTargetActor_Radius，用于选择目标数据
 UAbilityTask_WaitTargetData* TargetDataTask = UAbilityTask_WaitTargetData::WaitTargetData(this, TargetingConfirmation::Instant, TargetActor);
 TargetDataTask->ValidData.AddDynamic(this, &UMyGameplayAbility::OnTargetDataReceived);
```

`TargetingConfirmation` 通常包含以下几种类型：
- **Instant**：立即确认目标数据，无需玩家进一步操作；
- **UserConfirmed**：等待玩家确认目标数据，例如通过点击确认按钮；
- **Custom**：自定义确认机制，通常需要额外的逻辑来处理确认；
- **CustomMulti**；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240721023723.png)

4. `UAbilityTask_WaitMovementModeChange`：等待角色的移动模式（`Movement Mode`）发生变化；

```cpp
 UAbilityTask_WaitMovementModeChange* MovementTask = UAbilityTask_WaitMovementModeChange::CreateWaitMovementModeChange(this, EMovementMode::MOVE_Flying);
 MovementTask->OnChange.AddDynamic(this, &UMyGameplayAbility::OnMovementModeChanged);
```

改变角色移动模式：
```cpp
Character->GetCharacterMovement()->SetMovementMode(EMovementMode::MOVE_Walking);
```

5. `UAbilityTask_WaitConfirm`：等待玩家确认（如按下确认键）；

```cpp
 UAbilityTask_WaitConfirm* ConfirmTask = UAbilityTask_WaitConfirm::WaitConfirm(this);
 ConfirmTask->OnConfirm.AddDynamic(this, &UMyGameplayAbility::OnConfirm);
```

6. `UAbilityTask_WaitInputPress`：等待玩家按下输入键；

```cpp
 UAbilityTask_WaitInputPress* InputPressTask = UAbilityTask_WaitInputPress::WaitInputPress(this, true);
 InputPressTask->OnPress.AddDynamic(this, &UMyGameplayAbility::OnInputPress);
```

7. `UAbilityTask_WaitGameplayTag`：等待特定的游戏标签（`Gameplay Tag`）被添加或移除；

```cpp
 UAbilityTask_WaitGameplayTagAdded* TagAddedTask = UAbilityTask_WaitGameplayTagAdded::WaitGameplayTagAdd(this, Tag);
 TagAddedTask->Added.AddDynamic(this, &UMyGameplayAbility::OnTagAdded);
```

# 示例代码

以`WaitDelay`举例：

```cpp
// MyGameplayAbility.cpp

#include "MyGameplayAbility.h"
#include "AbilitySystemComponent.h"
#include "Abilities/Tasks/AbilityTask_WaitDelay.h"

void UMyGameplayAbility::ActivateAbility(const FGameplayAbilitySpecHandle Handle, const FGameplayAbilityActorInfo* ActorInfo, const FGameplayAbilityActivationInfo ActivationInfo, const FGameplayEventData* TriggerEventData)
{
    if (HasAuthorityOrPredictionKey(ActorInfo, &ActivationInfo))
    {
	    //创建任务
        UAbilityTask_WaitDelay* WaitTask = UAbilityTask_WaitDelay::WaitDelay(this, 2.0f);
        //绑定回调
        WaitTask->OnFinish.AddDynamic(this, &UMyGameplayAbility::OnWaitDelayFinish);
        //激活任务
        WaitTask->ReadyForActivation();
    }
}

void UMyGameplayAbility::OnWaitDelayFinish()
{
    // 延迟结束后的逻辑
}
```