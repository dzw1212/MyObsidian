虚幻自5.1起默认启用增强输入系统，代替之前的动作映射与轴映射；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134238.png)

# 定义IA

在`Input/Actions`目录下可以看到当前定义的`InputAction`；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134437.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134454.png)

在详细面板内，可以看到`IA`的类型；
IA_Move为2维数据类型，IA_Jump则为触发条件为Press的bool数据类型；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134747.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134820.png)


## ValueType的选择

- 1D Axis (float)
	- **适用场景**: 处理单一轴向的连续输入，如摇杆的上下或左右、扳机键的按压程度。
	- **示例**: 角色移动（前后）、车辆油门/刹车。

- 2D Axis (Vector2D)
	- **适用场景**: 处理二维平面上的输入，如摇杆的上下左右组合。
	- **示例**: 角色移动（前后左右）、相机视角控制。

- Float
	- **适用场景**: 处理单一浮点数值，通常用于1D Axis类似的情况。
	- **示例**: 扳机键的按压程度、滑块的数值变化。

- Boolean (bool)
	- **适用场景**: 处理简单的开关型输入，如按键的按下/松开。
	- **示例**: 跳跃、射击、交互。


# 配置IMC

在`Input`目录下，有负责映射`IA`的`Input Mapping Context`；`IA`需要在`IMC`中与具体的按键映射才能生效；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134957.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135009.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135110.png)

## IA_Move

通常情况下，按键D代表向右移动，对应x轴的正方向，这也是默认的Input操作，因此按键为D时什么也不做，按键为A时只需要进行一次反转；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210140558.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260103000307.png)


![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210140705.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209174615.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209174649.png)

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251228230729.png)

## IA_Look

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251228225318.png)

启用Pitch：
	弹簧臂：
![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251228223130.png)

如果希望禁止人物跟着Yaw旋转：
	Movement组件：
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251228225100.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251228225247.png)


# 启用

![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135534.png)

如果是使用代码：

```cpp
#include "EnhancedInputSubsystems.h"

void AMyPlayerController::BeginPlay()
{
	Super::BeginPlay();

	check(AuraInputMappingContext);

	auto EnhancedInputSubsystem = ULocalPlayer::GetSubsystem<UEnhancedInputLocalPlayerSubsystem>(GetLocalPlayer());
	if (EnhancedInputSubsystem)
	{
		EnhancedInputSubsystem->AddMappingContext(AuraInputMappingContext, 0);
	}
}
```


绑定InputAction与对应的处理接口：

```cpp
void AAuraPlayerController::SetupInputComponent()
{
	Super::SetupInputComponent();

	auto EnhancedInputComponent = CastChecked<UEnhancedInputComponent>(InputComponent);
	EnhancedInputComponent->BindAction(AuraInputAction, ETriggerEvent::Triggered, this, &AAuraPlayerController::Move);
}

void AAuraPlayerController::Move(const FInputActionValue& InputActionValue)
{
	auto InputAxisVector = InputActionValue.Get<FVector2D>();

	FRotator YawRotation(0.f, GetControlRotation().Yaw, 0.f);
	FVector ForwardDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::X);
	FVector RightDirection = FRotationMatrix(YawRotation).GetUnitAxis(EAxis::Y);

	auto ControlledPawn = GetPawn<APawn>();
	if (!ControlledPawn)
		return;

	ControlledPawn->AddMovementInput(ForwardDirection, InputAxisVector.Y);
	ControlledPawn->AddMovementInput(RightDirection, InputAxisVector.X);
}
```

*之所以可以直接将InputComponent转为EnhancedInputComponent，是因为项目设置->输入下：*

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209185005.png)


# 优先级

事件的传递使分层的，并且具有优先级；
在复杂的游戏场景中可以使用多个`IMC`，并通过优先级控制事件传递顺序；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135729.png)

如果想要`IA`不传递到下一个层级，可以通过`Consume Input`来控制；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135859.png)

# 绑定按键与功能

```cpp
FInputActionBinding& BindAction(
    const UInputAction* Action, // 输入动作
    ETriggerEvent TriggerEvent, // 触发事件类型
    UObject* Object, // 回调函数所属的对象
    FName FunctionName // 回调函数的名称
);
```

## 触发事件类型

- `Triggered`: 输入动作被触发（例如按下按键）
	- 在输入动作的整个持续期间可能会**多次触发**（取决于输入设备的采样率）；
	- 适合用于连续事件或需要实时响应的操作，如连续发射子弹；

- `Started`: 输入动作刚开始时触发（例如按键刚刚按下）
	- 只在输入动作的起始时刻**触发一次**；
	- 适用于一次性事件；

- `Ongoing`: 输入动作持续中（例如按住按键）
	- 在输入动作的持续期间，每帧都会触发一次；
	- 与 `Triggered` 不同，`Ongoing` 更强调“持续状态”而不是“触发事件”；
	- 比如按住瞄准键时持续调整摄像机视角、按住加速键时持续增加车辆速度；

- `Canceled`: 输入动作被取消（例如松开按键）
	- 在输入动作被中断或异常结束时触发；
	- 适合用于处理异常情况或取消逻辑；

- `Completed`: 输入动作完成（例如松开按键）
	- 只在输入动作正常结束时**触发一次**；
	- 适合用于完成操作或清理逻辑；

