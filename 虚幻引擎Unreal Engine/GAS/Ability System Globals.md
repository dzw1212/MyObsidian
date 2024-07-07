能力系统全局类（`AbilitySyetemGlobals`）类中保存有关`GAS`的全局信息，大多数变量可以通过`DefaultGame.ini`进行设置，一般情况下无需与该类进行交互，但是如果需要子类化`GameplayCueManager`或`GameplayEffectContext`，则必须通过`AbilitySystemGlobals`来完成；

如果要对`AbilitySystemGlobals`子类化，也需要在`DefaultGame.ini`中配置：
```cpp
[/Script/GameplayAbilities.AbilitySystemGlobals]
AbilitySystemGlobalsClassName="/Script/ParagonAssets.PAAbilitySystemGlobals"
```

