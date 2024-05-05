在`GAS`中，`Gameplay Cue`是一个系统，用于响应游戏中的事件，以触发视觉、音频或游戏逻辑效果。这些事件通常与游戏中的特定状态或能力的激活、应用或移除有关。

`GC`可以看作是一种机制，允许开发者将游戏逻辑与表现层分离。例如，当一个角色施放一个火球技能时，技能的逻辑处理（如伤害计算、状态应用等）由`GAS`的其他部分（如`Gameplay Abilities`）处理，而火球的视觉效果、声音和任何相关的环境影响则可以通过`GC`来触发。

`GC`通常通过`Gameplay Events`触发，这些事件可以由`Gameplay Abilities`、`Gameplay Effects`或其他游戏逻辑发出。开发者可以创建`GC`的实现，来定义当特定事件发生时应该发生什么，这包括播放动画、生成粒子效果、播放声音等。

# GC标签

1. 一个`GC`标签必须以`GameplayCue.`为前缀，如`GameplayCue.Character.Health.Increase`；
2. 在使用GC标签之前，必须在项目的`GameplayTags`数据库中注册该`GC`标签；
3. 对于每个`GC`标签，都需要一个响应的`GameplayCueNotify`类，并且这个类能正确响应`GC`触发时的所有情况（如激活、持续和移除等）；

# GC通知

`GameplayCueNotify` 是一种用于响应游戏中的特定事件或状态变化的机制，主要用于处理那些需要在游戏中快速响应的视觉效果、声音效果或其他游戏逻辑；

主要有两种形式：
1. **GameplayCueNotify_Static**：这是一个不包含蓝图逻辑的轻量级类，主要用于处理简单的、不需要维护状态的响应，如播放一个声音或粒子效果；

2. **GameplayCueNotify_Actor**：这是一个更复杂的类，它派生自 `AActor`。它可以包含更复杂的逻辑和状态，适用于需要交互或持续一段时间的效果，如持续的环境效果或者是可以与玩家互动的对象。

# 触发GC

例如有一个治疗效果，你可能会在这个 `GameplayEffect` 中包含一个 `GameplayTag`，如 `GameplayCue.Health.Increase`，当效果应用时，系统会查找并触发与这个标签关联的所有 `GameplayCueNotify` 对象，从而在游戏中产生相应的视觉和声音效果；

也可以通过`ASC`直接触发`GC`，有以下函数：
```cpp
/** GameplayCues can also come on their own. These take an optional effect context to pass through hit result, etc */
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void ExecuteGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** Add a persistent gameplay cue */
void AddGameplayCue(const FGameplayTag GameplayCueTag, FGameplayEffectContextHandle EffectContext = FGameplayEffectContextHandle());
void AddGameplayCue(const FGameplayTag GameplayCueTag, const FGameplayCueParameters& GameplayCueParameters);

/** Remove a persistent gameplay cue */
void RemoveGameplayCue(const FGameplayTag GameplayCueTag);
	
/** Removes any GameplayCue added on its own, i.e. not as part of a GameplayEffect. */
void RemoveAllGameplayCues();
```

# GC管理器

在`GameplayCueManager` 是一个核心组件，负责管理和调度所有与 `GameplayCues` 相关的操作；它是一个全局管理器，用于处理 `GameplayCue` 的注册、触发和执行；

`GameplayCueManager` 维护一个关于所有 `GameplayCues` 和对应 `GameplayCueNotify` 类的映射。这使得当一个 `GameplayCue` 被触发时，`GameplayCueManager` 可以快速找到并执行相应的响应逻辑；

默认情况下，`GameplayCueManager` 会扫描整个游戏目录以查找 `GameplayCueNotifies` 并在游戏时将其加载到内存中。我们可以通过在 `DefaultGame.ini` 中设置来更改扫描的路径：
```cpp
[/Script/GameplayAbilities.AbilitySystemGlobals]
GameplayCueNotifyPaths="/Game/GASDocumentation/Characters"
```

如果出于性能考虑，希望异步加载`GC`，可以创建`GameplayCueManager`的子类，并覆写`ShouldAsyncLoadRuntimeObjectLibraries`函数：
```cpp
virtual bool ShouldAsyncLoadRuntimeObjectLibraries() const override
{
	return false;
}
```

指定使用新建的`GC`管理器子类：
```cpp
[/Script/GameplayAbilities.AbilitySystemGlobals]
GlobalGameplayCueManagerClass="/Script/ParagonAssets.PBGameplayCueManager"
```