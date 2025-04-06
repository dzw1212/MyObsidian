`GEEC`（游戏效果执行计算，`Gameplay Effect Execution Calculation`），是`GameplayEffect`对`ASC`进行更改的最强大的方法；类似于[[修饰符幅度计算 MMC]]，它也可以捕获属性并对其进行快照，但不同的是`GEEC`能够更改多个属性，并且更加灵活，与此同时牺牲的是可预测性；

`GEEC`仅可用于瞬时和周期性的`GameplayEffect`；

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308000210.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308013215.png)

# 使用方式

## 继承并实现Execution接口

```cpp
// MyDamageExecutionCalc.h
#pragma once

#include "CoreMinimal.h"
#include "GameplayEffectExecutionCalculation.h"
#include "MyDamageExecutionCalc.generated.h"

UCLASS()
class MYGAME_API UMyDamageExecutionCalc : public UGameplayEffectExecutionCalculation
{
    GENERATED_BODY()

public:
    UMyDamageExecutionCalc();

    virtual void Execute_Implementation(const FGameplayEffectCustomExecutionParameters& ExecutionParams, FGameplayEffectCustomExecutionOutput& OutExecutionOutput) const override;
};
```

继承基类`UGameplayExecutionCalculation`内的虚函数：

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308140214.png)

## 捕获属性

在[[修饰符幅度计算 MMC]]中，属性是通过`FGameplayEffectAttributeCaptureDefinition`来捕获的：

![650](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308143135.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308143201.png)

GEEC中也是通过类似的做法，不过GEEC提供了一些宏来帮忙捕获属性：

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308150032.png)

对应关系是：
![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308151229.png)

`DECLARE`然后`DEFINE`之后，同样还需要添加进`RelevantAttributesToCapture`中；

## 添加Modifier

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308155013.png)

---

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308155212.png)

# GameplayEffect中指定GEEC

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250308155541.png)

