在`GAS`中，`GameplayAttribute`是一个核心概念，用于表示、存储和管理游戏中角色或对象的各种属性（如生命值、法力值、速度等）。这些属性是游戏逻辑和游戏玩法中不可或缺的部分，它们可以被修改和查询，以支持复杂的游戏能力、效果和交互。

# 核心特点

- **数据驱动**：`GameplayAttribute`通常与数据表（如`UDataTable`）结合使用，允许游戏设计师通过配置而非硬编码的方式定义和修改属性。
- **高度可扩展**：开发者可以根据需要自定义和扩展属性，以支持游戏的特定需求。
- **与GameplayEffect交互**：在`GAS`中，`GameplayEffect`用于修改`GameplayAttribute`，例如，一个“治疗”效果可能会增加角色的生命值属性。
- **网络复制**：`GAS`设计用于多人游戏，因此`GameplayAttribute`支持网络复制，确保所有玩家客户端的游戏状态同步。

# 实现方式

`GameplayAttribute`通过C++类`FGameplayAttribute`实现。开发者需要在自己的代码中定义具体的属性，通常是通过继承`UAttributeSet`类来实现的。`UAttributeSet`类为属性提供了存储、管理和复制的功能。

# 示例

假设你正在开发一个有生命值（`Health`）和法力值（`Mana`）的角色，你可能会这样定义你的`AttributeSet`：

```cpp
#include "AbilitySystemComponent.h"
#include "AttributeSet.h"
#include "GameplayTagContainer.h"
#include "YourAttributeSet.generated.h"

UCLASS()
class YOURGAME_API UYourAttributeSet : public UAttributeSet
{
    GENERATED_BODY()

public:
    UYourAttributeSet();

    // 使用GAMEPLAYATTRIBUTE_PROPERTY_GETTER宏来声明属性的Getter函数
    // 生命值
    UPROPERTY(BlueprintReadOnly, Category = "Attributes|Health")
    FGameplayAttributeData Health;
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UYourAttributeSet, Health)

    // 法力值
    UPROPERTY(BlueprintReadOnly, Category = "Attributes|Mana")
    FGameplayAttributeData Mana;
    GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UYourAttributeSet, Mana)

    // 你可以继续添加更多属性...
};
```

在这个例子中，`Health`和`Mana`属性被定义为`FGameplayAttributeData`类型，这是`GAS`用于存储属性值的标准方式。通过在`UAttributeSet`派生类中定义这些属性，你可以在游戏中使用`GAS`的功能来修改和查询这些值。

# GETTER与SETTER宏

使用宏来定义`GameplayAttribute`的getter和setter方法是一种简化代码和提高可维护性的技术。这些宏背后的原理是预处理器指令，它们在编译时会展开成实际的代码，从而减少了手动编写重复代码的需要。

假设我们有一个宏定义如下：

```cpp
#define GAMEPLAYATTRIBUTE_PROPERTY_GETTER(ClassName, PropertyName) \
    FORCEINLINE const FGameplayAttributeData* Get##PropertyName##() const \
    { \
        return &PropertyName; \
    }
```

当你在`UAttributeSet`派生类中使用这个宏时：

```cpp
GAMEPLAYATTRIBUTE_PROPERTY_GETTER(UYourAttributeSet, Health)
```

预处理器会将它展开为：

```cpp
FORCEINLINE const FGameplayAttributeData* GetHealth() const
{
    return &Health;
}
```

- **减少重复代码**：对于每个属性的getter和setter，如果手动编写，代码会非常相似，只是属性名不同。使用宏可以避免这种重复，使代码更加简洁。
- **易于维护**：如果getter和setter的通用模式需要更改，你只需修改宏的定义，而不是每个地方的实现。这使得维护和更新变得更加容易。
- **提高一致性**：宏确保了所有属性的getter和setter遵循相同的模式，减少了因手动编码错误而引入的不一致性。

---

## GAMEPLAYATTRIBUTE_VALUE_GETTER

`GAMEPLAYATTRIBUTE_VALUE_GETTER`宏用于创建一个函数，该函数返回属性的当前值。这个宏使得访问属性值变得简单直接，无需直接与`FGameplayAttributeData`结构交互。

```cpp
GAMEPLAYATTRIBUTE_VALUE_GETTER(AttributeName)
```

这将生成一个名为`GetAttributeName()`的函数，用于返回`AttributeName`属性的当前值。

## GAMEPLAYATTRIBUTE_VALUE_SETTER

`GAMEPLAYATTRIBUTE_VALUE_SETTER`宏用于创建一个函数，该函数允许你设置属性的当前值。这是修改属性值的标准方法，确保所有相关的游戏逻辑和网络复制都能正确处理属性值的变化。

```cpp
GAMEPLAYATTRIBUTE_VALUE_SETTER(AttributeName)
```

这将生成一个名为`SetAttributeName(float NewValue)`的函数，用于设置`AttributeName`属性的当前值。

## GAMEPLAYATTRIBUTE_VALUE_INITTER

`GAMEPLAYATTRIBUTE_VALUE_INITTER`宏用于创建一个函数，该函数用于初始化属性的值。这通常在属性集（Attribute Set）被创建时使用，以确保所有属性都有一个明确的初始值。

```cpp
GAMEPLAYATTRIBUTE_VALUE_INITTER(AttributeName)
```

这将生成一个名为`InitAttributeName(float InitialValue)`的函数，用于初始化`AttributeName`属性的值。