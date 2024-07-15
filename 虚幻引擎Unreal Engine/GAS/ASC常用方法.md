在虚幻引擎的Gameplay Ability System (GAS) 中，`UAbilitySystemComponent` 是一个核心组件，用于管理和执行能力（Abilities）和效果（Effects）。以下是一些常用的方法：

1. **Ability 相关**
	- `GiveAbility(FGameplayAbilitySpec AbilitySpec)`: 给组件添加一个能力。

	- `ClearAbility(FGameplayAbilitySpecHandle Handle)`: 移除一个特定的能力。

	- `TryActivateAbility(FGameplayAbilitySpecHandle Handle, bool bAllowRemoteActivation)`: 尝试激活一个特定的能力。

	- `CancelAbility(FGameplayAbilitySpecHandle Handle)`: 取消一个正在进行的能力；

	- `InitAbilityActorInfo(AActor* InOwnerActor, AActor* InAvatarActor)`: 用于初始化与能力系统相关的角色信息，通常在角色或玩家状态初始化时或`PlayerController`的`OnPossess`时调用，以确保能力系统组件正确地知道它所关联的角色和控制器；

	- `BindAbilityActivationToInputComponent(UInputComponent* InputComponent, FGameplayAbilityInputBinds BindInfo)`：将输入组件与能力系统组件绑定，从而使得玩家可以通过输入来激活或取消能力；


2. **Effect 相关**
   - `ApplyGameplayEffectToSelf(TSubclassOf<UGameplayEffect> GameplayEffectClass, float Level, FGameplayEffectContextHandle EffectContext)`: 将一个效果应用到自己身上。

   - `ApplyGameplayEffectToTarget(TSubclassOf<UGameplayEffect> GameplayEffectClass, UAbilitySystemComponent* Target, float Level, FGameplayEffectContextHandle EffectContext)`: 将一个效果应用到目标身上。

   - `ApplyGameplayEffectSpecToSelf/ApplyGameplayEffectSpecToTarget`: 相比非`spec`的版本，可以在应用效果之前自定义效果的规格；



   - `RemoveActiveGameplayEffect(FActiveGameplayEffectHandle Handle)`: 移除一个激活的效果。

   - `MakeEffectContext()`: 创建并返回一个 `FGameplayEffectContextHandle` 对象，包含了关于`Effect`应用的上下文信息，例如施加者、目标、命中结果等；

   - `MakeOutgoingSpec(TSubclassOf<UGameplayEffect> GameplayEffectClass, float Level, FGameplayEffectContextHandle EffectContext)` : 创建并返回一个 `FGameplayEffectSpecHandle` 对象，包含了关于即将应用的`Effect`的详细信息，例如等级、持续时间、修改器等；





3. **Attribute 相关**
   - `GetNumericAttribute(FGameplayAttribute Attribute)`: 获取一个属性的数值。

   - `SetNumericAttributeBase(FGameplayAttribute Attribute, float NewBaseValue)`: 设置一个属性的基础数值。

   - `GetGameplayAttributeValueChangeDelegate(FGameplayAttribute Attribute)`: 获取一个属性值变化的委托。



4. **Tag 相关**
	- `AddLooseGameplayTag(FGameplayTag Tag)`: 添加一个松散的Gameplay Tag。

	- `RemoveLooseGameplayTag(FGameplayTag Tag)`: 移除一个松散的Gameplay Tag。

	- `HasMatchingGameplayTag(FGameplayTag TagToCheck)`: 检查是否有匹配的Gameplay Tag。

	- `RegisterGameplayTagEvent(FGameplayTag Tag, EGameplayTagEventType::Type EventType=EGameplayTagEventType::NewOrRemoved)`: 注册一个委托，当指定的标签发生变化时，该委托会被调用；

	- `UnregisterGameplayTagEvent(FDelegateHandle DelegateHandle, FGameplayTag Tag, EGameplayTagEventType::Type EventType=EGameplayTagEventType::NewOrRemoved)`: 取消某个委托的注册；

	- `SetTagMapCount(const FGameplayTag& Tag, int32 NewCount)`: 直接设置指定标签的计数值，标签计数用于管理标签的状态。例如，当标签计数大于零时，标签被视为“激活”状态；当计数为零时，标签被视为“非激活”状态。



5. **Cooldowns 与 Costs 相关**
	- `GetCooldownTimeRemaining(FGameplayAbilitySpecHandle Handle)`: 获取一个能力的剩余冷却时间。

	- `GetCurrentAbilityCost(FGameplayAbilitySpecHandle Handle)`: 获取一个能力的当前消耗。


6. **网络同步相关**
	- `SetIsReplicated(bool bShouldReplicate)`: 设置组件是否应该被复制；

	- `SetReplicationMode(EGameplayEffectReplicationMode NewReplicationMode)`: 设置组件的复制模式，复制模式决定了组件如何以及何时被复制；
		`Minimal`: 仅复制最少量的数据；
		`Mixed`: 复制部分数据；
		`Full`: 复制所有数据

