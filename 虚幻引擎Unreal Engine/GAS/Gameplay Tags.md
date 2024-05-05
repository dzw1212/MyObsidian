`Unreal Engine`中的`Gameplay Tags`系统是一种用于标识、分类和组织游戏内容和行为的工具。它允许开发者通过标签（`Tags`）来标记和查询游戏中的对象和事件，从而实现更灵活和可扩展的游戏设计和代码管理。

`Gameplay Tags`是一种**层次化**的字符串标签，可以被赋予游戏中的各种元素，比如角色、物品、技能等。这些标签遵循一定的命名规则，通常以“**父标签.子标签**”的形式组织，形成一个**树状的层次结构**。这种结构不仅便于管理和扩展标签系统，还可以通过父标签来高效地查询和筛选具有相关子标签的对象。

例如，你可以有一个名为“武器.枪械.手枪”的`Gameplay Tag`，这样你就可以轻松地查询所有的“枪械”或者更具体的“手枪”标签的物品。

`Gameplay Tags`的主要用途包括：

- **分类和过滤**：通过标签来快速识别和筛选游戏中的对象和事件，例如查找所有的“敌人”或“可破坏物”。
- **游戏逻辑**：使用标签来控制游戏逻辑的流程，例如只允许“魔法”标签的技能穿透“魔法盾”标签的防御。
- **数据驱动设计**：通过标签来驱动游戏内容的配置和行为，使得游戏设计更加灵活和可配置。

使用`Gameplay Tags`系统，开发者可以构建出更加模块化和可重用的游戏逻辑，提高游戏开发的效率和灵活性。

# 在GAS中的使用

在给对象赋予标签时，如果对象拥有`ASC`，通常会将标签加在`ASC`上，以便于`GAS`交互，`UAbilitySystemComponent`中实现了`IGameplayTagAssetInterface`，提供了访问标签的接口；

`FGameplayTagContainer`中可以存储多个`FGameplayTag`，这种做法比使用数组来管理标签更为高效（如`TArray<FGameplayTag>`）；如果项目设置中启用了`Fast Replication`功能，`FGameplayTagContainer`就能更高效地打包和复制标签，这有利于服务器与客户端之间的标签同步；

## 标签的同步

如果一个标签是通过`Gameplay Effect`添加的，那它会自动被复制并同步给所有客户端；

如果你想手动管理标签的同步，可以使用`LooseGameplayTags`，这种标签不会被复制；

## 获取标签

获取标签的引用：
```cpp
FGameplayTag::RequestGameplayTag(FName("Your.GameplayTag.Name"))
```

获取标签的父级或子级（需要用到`GameplayTagManager`）：
```cpp
//获取父标签
TArray<FGameplayTag> ParentTags;
ChildTag.RequestGameplayTagParents(ParentTags);

//获取子标签
UGameplayTagsManager& TagManager = UGameplayTagsManager::Get();
TArray<FGameplayTag> ChildTags;
TagManager.RequestGameplayTagChildren(ParentTag, ChildTags);
```

## 标签事件

`ASC`可以通过`RegisterGameplayTagEvent`指定标签发生更改时的回调函数，并且通过`EGameplayTagEventType`来筛选标签事件类型；

```cpp
#include "GameplayTagContainer.h"
#include "AbilitySystemComponent.h"

void UMyClass::SetupTagEventListeners(UAbilitySystemComponent* AbilitySystemComp)
{
    FGameplayTag SomeGameplayTag = FGameplayTag::RequestGameplayTag(FName("Character.State.Poisoned"));
    
    // 订阅标签添加或移除的事件
    AbilitySystemComp->RegisterGameplayTagEvent(SomeGameplayTag, EGameplayTagEventType::NewOrRemoved).AddUObject(this, &UMyClass::OnTagChanged);
}

void UMyClass::OnTagChanged(const FGameplayTag Tag, int32 NewCount)
{
    if (NewCount > 0)
    {
        UE_LOG(LogTemp, Log, TEXT("Tag %s added"), *Tag.ToString());
        // 处理标签添加的逻辑
    }
    else
    {
        UE_LOG(LogTemp, Log, TEXT("Tag %s removed"), *Tag.ToString());
        // 处理标签移除的逻辑
    }
}
```

# 标签配置文件

`Gameplay Tag`系统的配置通常是通过一个名为`DefaultGameplayTags.ini`的配置文件进行的。这个文件位于项目的`Config`目录下。通过编辑这个文件，你可以定义和组织你的游戏中使用的所有`Gameplay Tags`。

文件路径示例：
```
[YourProjectDirectory]/Config/DefaultGameplayTags.ini
```

在`DefaultGameplayTags.ini`文件中，你可以添加、修改或删除`Gameplay Tags`，以及配置它们的层次结构。这个文件的结构允许你以树状形式组织标签，例如：

```ini
+GameplayTags=(TagName="Character.Hero", Comment="Tag for hero characters")
+GameplayTags=(TagName="Character.Villain", Comment="Tag for villain characters")
+GameplayTags=(TagName="Weapon.Sword", Comment="Tag for swords")
+GameplayTags=(TagName="Weapon.Gun", Comment="Tag for guns")
```

这里的`+GameplayTags=`条目用于添加新的标签，`TagName`指定了标签的完整名称，而可选的`Comment`字段则为标签提供了描述。

# 标签编辑器

除了手动编辑`DefaultGameplayTags.ini`文件外，UE的编辑器也提供了一个可视化的`Gameplay Tag`编辑器，允许你在不直接编辑配置文件的情况下管理标签。你可以在编辑器中通过导航到`Edit` -> `Project Settings` -> `Gameplay Tags`来访问和配置这些标签。

通过使用`Gameplay Tag`编辑器，你可以更加直观地添加、编辑和组织你的标签，同时也可以查看和管理标签的层次结构。编辑器中的改动会自动更新到`DefaultGameplayTags.ini`文件中，确保了配置的一致性和可追踪性。