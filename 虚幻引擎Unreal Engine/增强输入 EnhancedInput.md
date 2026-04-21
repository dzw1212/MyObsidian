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

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260223141320.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210140705.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209174615.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209174649.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260223134801.png)


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

# 用户自定义按键

想要实现一个完美的自定义改键菜单，**`Player Mappable Key Settings`** 和 **`InputKeySelector`** 是完美搭档。

简单来说：前者是底层逻辑的“身份证”，后者是前端 UI 的“捕获器”。

## Player Mappable Key Settings（底层数据身份证）

这是配置在你的 `InputAction`（输入操作，比如 `IA_Interact`）资产内部的一个设置面板。它的核心作用是告诉引擎：**“这个按键允许玩家修改，并且我要给它建档保存。”**

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260223225806.png)

双击打开你的 `IA_Interact`，勾选并展开这个设置后，你会看到几个关键参数：

1. **Name (映射名称)**
* **这是最核心的参数！** 它是一个 `FName`（比如填入 `InteractKey`）。
* 它是这个按键在底层配置文件（`.ini`）中的**唯一 ID**。以后玩家改键保存、读取按键，引擎全靠这个名字来找人。如果不填这个名字，引擎的设置系统（User Settings）就不会接管它。


2. **Display Name (显示名称)**
* 这是给玩家看的本地化文本（Text），比如你可以填入“交互”或“开门”。
* 在生成设置菜单时，你可以直接读取这个变量显示在屏幕上，不需要在 UI 里手动打字，方便多语言翻译。


3. **Display Category (显示类别)**
* 用来在设置菜单里分类。比如你可以填入“移动”、“战斗”、“交互”。以后在 UI 遍历按键时，可以根据这个把它们分门别类地显示出来。

## InputKeySelector 控件（前端交互捕获器）

这是 UMG（UI 系统）里专门为改键量身定制的控件。

对于非专业 UI 程序来说，自己写一个“点击按钮 -> 拦截键盘输入 -> 获取按下的键”是非常痛苦的，因为你很容易不小心触发了游戏的“跳跃”或者打开了“暂停菜单”。`InputKeySelector` 直接把这些全封装好了。

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260223230015.png)


1. **核心行为逻辑**
* 当玩家点击这个控件时，它会自动进入“监听状态”（内部通常会显示类似“请按下一个键...”的提示）。
* 在监听状态下，它会**拦截所有的游戏输入**。此时玩家按下任何键（键盘字母、鼠标侧键、手柄摇杆点击），它都会捕获这个按键，更新自己的显示，然后立刻退出监听状态。

1. **核心事件 (Events)**
* **`On Key Selected` (按键选定后)**：当玩家成功输入了一个新按键时触发。它会输出一个 `Selected Key`（玩家按下的键）。这里就是你连接底层改键代码的地方。
* **`On Is Selecting Key Changed` (选择状态改变时)**：当控件进入或退出监听状态时触发。你可以用它来播放音效，或者让背景变暗变亮。

3. **关键细节设置 (Details Panel)**
* **`Allow Modifier Keys` (允许修饰键)**：如果勾选，玩家可以设置 `Ctrl + E` 或 `Shift + F` 这种组合键。
* **`Allow Gamepad Keys` (允许手柄按键)**：是否允许捕获手柄输入。
* **`Escape Keys` (取消按键)**：通常设置为 `Esc` 键。当控件处于监听状态时，玩家按下 `Esc`，它会放弃本次修改，恢复到原来的按键。

## 协同工作

当你在做一个设置界面（比如 `WBP_Settings`）时，标准的闭环流程是这样的：

1. **初始化 (UI 刚打开时)**
* 找到增强输入子系统的 User Settings。
* 通过 `Player Mappable Key Settings` 里配置的那个唯一 **Name**（比如 `InteractKey`），查询玩家当前用的是什么键。
* 拿到当前的键后，调用 `InputKeySelector` 的 **`Set Selected Key`** 节点，让它一显示出来就正确显示当前的按键（比如 "E"）。

1. **修改 (玩家操作 UI 时)**
* 玩家点击 `InputKeySelector`，按下了 "F" 键。
* `InputKeySelector` 触发 **`On Key Selected`** 事件，输出 "F"。

1. **保存 (数据写回底层)**
* 在事件节点后，再次调用 User Settings，使用 **`Map Player Key`** 节点。
* 将 "F" 绑定到那个唯一的 **Name** (`InteractKey`) 上。
* 调用 **`Save Settings`**，引擎自动把 "交互=F" 写入本地硬盘。

## 添加自定义配置

### 1. 启用配置

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260223233217.png)

### 2. 读取Player Mapped Key Settings

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260223235058.png)

### 3. 添加Map Player Key

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260223235303.png)


### 4. 应用与保存配置

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260223235402.png)

本地存储文件：
![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260223233144.png)


# 与CommonUI协同

*https://dev.epicgames.com/documentation/zh-cn/unreal-engine/using-commonui-with-enhnaced-input-in-unreal-engine*

# 监听硬件输入切换

CommonUI将 UI 与输入的绑定逻辑完全数据化（Data-Driven）了。你不再需要写任何“检测按键 -> 替换图片”的蓝图代码，一切通过配置数据资产（Data Asset）自动完成。

[[Common UI]]
## 1. CommonUI启动EnhancedInput支持

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260224004615.png)

## 2. 视口类设置为CommonGameViewportClient

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260224154220.png)


## 3. 创建Controller Data并配置

这是最核心的一步。我们需要给键鼠和手柄各建一本“字典”，告诉引擎“键盘的 E 键长什么样”、“手柄的 RT 键长什么样”。

基于`CommonInputBaseControllerData`：

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260224181736.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260224182125.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260224182207.png)

配置到Common Input Setting中：
![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260224182342.png)

## 4. 创建CommonActionWidget

基于`CommonActivatableWidget`创建一个WBP，里面添加一个CommonActionWidget；

### 重写Input Config

CommonUI的默认行为是切换游戏模式为`Menu`，如果只是按键交互的话需要修改；

![image.png](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260224183439.png)

### 指定进度条材质参数与IA

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260224183621.png)

