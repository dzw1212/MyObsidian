`Unreal Engine`中的`Gameplay Tags`系统是一种用于标识、分类和组织游戏内容和行为的工具。它允许开发者通过标签（`Tags`）来标记和查询游戏中的对象和事件，从而实现更灵活和可扩展的游戏设计和代码管理。

`Gameplay Tags`是一种**层次化**的字符串标签，可以被赋予游戏中的各种元素，比如角色、物品、技能等。这些标签遵循一定的命名规则，通常以“**父标签.子标签**”的形式组织，形成一个**树状的层次结构**。这种结构不仅便于管理和扩展标签系统，还可以通过父标签来高效地查询和筛选具有相关子标签的对象。

例如，你可以有一个名为“武器.枪械.手枪”的`Gameplay Tag`，这样你就可以轻松地查询所有的“枪械”或者更具体的“手枪”标签的物品。

`Gameplay Tags`的主要用途包括：

- **分类和过滤**：通过标签来快速识别和筛选游戏中的对象和事件，例如查找所有的“敌人”或“可破坏物”。
- **游戏逻辑**：使用标签来控制游戏逻辑的流程，例如只允许“魔法”标签的技能穿透“魔法盾”标签的防御。
- **数据驱动设计**：通过标签来驱动游戏内容的配置和行为，使得游戏设计更加灵活和可配置。

使用`Gameplay Tags`系统，开发者可以构建出更加模块化和可重用的游戏逻辑，提高游戏开发的效率和灵活性。

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

除了手动编辑`DefaultGameplayTags.ini`文件外，UE的编辑器也提供了一个可视化的`Gameplay Tag`编辑器，允许你在不直接编辑配置文件的情况下管理标签。你可以在编辑器中通过导航到`Edit` -> `Project Settings` -> `Gameplay Tags`来访问和配置这些标签。

通过使用`Gameplay Tag`编辑器，你可以更加直观地添加、编辑和组织你的标签，同时也可以查看和管理标签的层次结构。编辑器中的改动会自动更新到`DefaultGameplayTags.ini`文件中，确保了配置的一致性和可追踪性。