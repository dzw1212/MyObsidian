在`GAS`中，`AbilitySystemComponent`是一个核心组件，它是整个系统运作的基础。这个组件负责管理和执行与角色相关的各种游戏能力（`Gameplay Abilities`），以及处理状态效果（`Gameplay Effects`）和属性（`Attributes`）。

`ASC`负责处理所有交互相关的功能，任何想要使用`GA`、拥有`Attribute`或者希望能接收`GE`的`Actor`都需要拥有`ASC`；

一般来说，开发者可以创建`ASC`的子类（但不强制）；

# 主要功能

- **管理游戏技能（Gameplay Abilities）**：`AbilitySystemComponent`负责激活、执行和终止游戏能力。它处理能力的冷却时间、成本和条件判断（例如，一个角色是否有足够的资源来使用某个能力）。

- **处理状态效果（Gameplay Effects）**：这些效果可以修改角色的属性值，如生命值、能量等。`AbilitySystemComponent`负责应用、追踪和移除这些效果。

- **属性管理**：它使用`AttributeSet`类来定义和存储角色的属性（例如，生命值、力量、敏捷等）。这些属性可以被游戏能力和状态效果读取和修改。

- **事件和代理**：`AbilitySystemComponent`提供了一套事件系统和代理（Delegates），允许游戏能力和其他系统在特定事件发生时响应，如属性变化、能力激活等。

# 实现方式

在游戏项目中，通常会在需要使用`GAS`的角色（如玩家角色或AI）上附加`AbilitySystemComponent`。然后，通过在这个组件上注册`AttributeSets`和定义的游戏能力，来为角色配置其能力和属性。

# 示例

假设有一个角色类`MyCharacter`，它继承自UE4的`ACharacter`类。要为这个角色添加`GAS`支持，你需要在其类定义中添加一个`AbilitySystemComponent`的实例，并确保它被正确初始化和配置。

```cpp
#include "AbilitySystemComponent.h"

class AMyCharacter : public ACharacter
{
    GENERATED_BODY()

public:
    AMyCharacter();

    // AbilitySystemComponent的引用
    UPROPERTY(VisibleAnywhere, BlueprintReadOnly, Category = "Abilities")
    UAbilitySystemComponent* AbilitySystemComponent;

    // 实现AbilitySystemInterface
    virtual UAbilitySystemComponent* GetAbilitySystemComponent() const override;
};

AMyCharacter::AMyCharacter()
{
    // 初始化AbilitySystemComponent
    AbilitySystemComponent = CreateDefaultSubobject<UAbilitySystemComponent>(TEXT("AbilitySystemComp"));
}

UAbilitySystemComponent* AMyCharacter::GetAbilitySystemComponent() const
{
    return AbilitySystemComponent;
}
```

# OwnerActor与AvatarActor

在`ASC`中，`OwnerActor`和`AvatarActor`是两个关键概念，它们有着明确的区别：

- **OwnerActor**：是指拥有该`AbilitySystemComponent`的实体。通常情况下，这是添加了ASC的那个角色或对象。在多数情况下，`OwnerActor`用于引用能力系统的“所有者”，比如一个玩家角色或者是一个NPC。它主要用于权限检查，确定哪个玩家控制该角色，以及在网络游戏中哪些操作需要复制。

- **AvatarActor**：是指实际执行能力（如施放技能、应用效果等）的实体。在许多情况下，`AvatarActor`和`OwnerActor`是同一个实体。然而，在某些情况下，它们可能会不同，比如当一个玩家控制一个召唤物或者是远程控制一个单位时，`OwnerActor`是玩家角色，而`AvatarActor`则是召唤物或远程控制的单位。这种区分允许系统灵活地处理能力的执行者和能力的所有者之间的关系。

如果`OwnerActor`和`AvatarActor`是不同的`Actor`，那么这两个`Actor``都应该实现IAbilitySystemInterface`。该接口有一个必须重载的函数，即 `UAbilitySystemComponent* GetAbilitySystemComponent() const`，它返回一个指向其 `ASC` 的指针，`ASC`通过查找该接口函数在系统内部进行交互。

# 附加在Actor还是PlayerState上

`PlayerState`是一个用于存储游戏中特定玩家相关信息的类，如玩家的分数、名称、团队信息等。它是游戏中每个玩家的状态的反映，并且在多人游戏中尤其重要，因为它包含了需要在客户端之间共享的玩家信息。

`ASC`通常会附加到`Character`或`PlayerState`上，这取决于开发者如何设计游戏架构。
当`ASC`附加到`PlayerState`时，意味着玩家的能力、属性和状态效果等信息是与玩家的状态直接关联的，而不是与玩家控制的角色或实体关联。这种设计允许玩家在游戏中需要重生、更换角色或实体时，保持其能力和属性不变。

如果将`ASC`附加在`PlayerState`上，那么就还需要增加 `PlayerState` 的 `NetUpdateFrequency`。该参数的默认值很低，可能会在客户端上发生属性和游戏标签等更改之前造成延迟或感觉滞后。请务必启用[[自适应网络更新频率]]（《堡垒之夜》中就使用了该功能）。

# 复制模式

`ASC`定义了三种不同的复制模式用于`Gameplay Effects`、`Gameplay Tags`、`Gameplay Cues`的复制（不包括`Gameplay Attributes`，其由`AttributeSet`负责复制），以适应不同的游戏类型和网络需求，这些复制模式分别是：

- **Full**：每个`Gameplay Effects`都复制给每个客户端；
- **Mixed**：`Gameplay Effects`复制给其拥有者客户端，`Gameplay Tags`和`Gameplay Cues`复制给所有人；
- **Minimal**：`Gameplay Effects`不复制给任何人，`Gameplay Tags`和`Gameplay Cues`复制给所有人；

# 创建与初始化

`ASC`的创建通常在其`OwnerActor`的构造函数中完成，并被标记为“已复制”，该过程只能在C++中完成；

如果报错`LogAbilitySystem: Warning: Can't activate LocalOnly or LocalPredicted ability %s when not local!`，则说明没有在客户端成功初始化`ASC`；

```cpp
AGDPlayerState::AGDPlayerState()
{
	// Create ability system component, and set it to be explicitly replicated
	AbilitySystemComponent = CreateDefaultSubobject<UGDAbilitySystemComponent>(TEXT("AbilitySystemComponent"));
	AbilitySystemComponent->SetIsReplicated(true);
	//...
}
```

---

- 对于 `ASC` 位于 `Pawn` 上的玩家控制角色，服务器上通常在通过 `Pawn` 的 `PossessedBy()` 函数（被`Controller`控制时）进行初始化，在客户端上通过 `PlayerController` 的 `AcknowledgePossession()` 函数进行初始化。

```cpp
//Server
void APACharacterBase::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	if (AbilitySystemComponent)
	{
		AbilitySystemComponent->InitAbilityActorInfo(this, this);
	}

	// ASC MixedMode replication requires that the ASC Owner's Owner be the Controller.
	SetOwner(NewController);
}
```

```cpp
//Client
void APAPlayerControllerBase::AcknowledgePossession(APawn* P)
{
	Super::AcknowledgePossession(P);

	APACharacterBase* CharacterBase = Cast<APACharacterBase>(P);
	if (CharacterBase)
	{
		CharacterBase->GetAbilitySystemComponent()->InitAbilityActorInfo(CharacterBase, CharacterBase);
	}

	//...
}
```

对于 `ASC` 位于 `PlayerState` 上的玩家控制角色，服务器上通常在 `Pawn` 的 `PossessedBy()` 函数中初始化，在客户端 `Pawn` 的 `OnRep_PlayerState()` 函数中初始化。这样可以确保 `PlayerState` 存在于客户端。

```cpp
// Server only
void AGDHeroCharacter::PossessedBy(AController * NewController)
{
	Super::PossessedBy(NewController);

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// Set the ASC on the Server. Clients do this in OnRep_PlayerState()
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// AI won't have PlayerControllers so we can init again here just to be sure. No harm in initing twice for heroes that have PlayerControllers.
		PS->GetAbilitySystemComponent()->InitAbilityActorInfo(PS, this);
	}
	
	//...
}
```

```cpp
// Client only
void AGDHeroCharacter::OnRep_PlayerState()
{
	Super::OnRep_PlayerState();

	AGDPlayerState* PS = GetPlayerState<AGDPlayerState>();
	if (PS)
	{
		// Set the ASC for clients. Server does this in PossessedBy.
		AbilitySystemComponent = Cast<UGDAbilitySystemComponent>(PS->GetAbilitySystemComponent());

		// Init ASC Actor Info for clients. Server will init its ASC when it possesses a new Actor.
		AbilitySystemComponent->InitAbilityActorInfo(PS, this);
	}

	// ...
}
```

# ActivableAbilities

在 `FGameplayAbilitySpecContainer ActivatableAbilities` 中，如果想要遍历 `ActivatableAbilities.Items`，请务必在循环上方添加 `ABILITYLIST_SCOPE_LOCK();`；作用域中的每个 `ABILITYLIST_SCOPE_LOCK(); `都会递增 `AbilityScopeLockCount`，并在其退出作用域时递减。请勿尝试在 `ABILITYLIST_SCOPE_LOCK();` 的作用域内移除能力（清除能力函数会在内部检查 `AbilityScopeLockCount`，以防止在列表被锁定时移除能力）。