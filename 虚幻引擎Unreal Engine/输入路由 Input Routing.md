输入路由（`Input Routing`）是一个复杂的系统，用于**管理和分发用户输入到适当的游戏对象或UI元素**。这个系统决定了哪个对象应该接收和响应特定的输入事件。以下是虚幻引擎输入路由系统的主要特点和概念：

1. 输入优先级
输入路由系统根据预定义的优先级规则来决定哪个对象应该首先接收输入。

2. 焦点系统
UI元素可以获得或失去焦点，影响它们是否接收键盘输入。

3. 命中测试
用于确定鼠标点击或触摸事件应该被哪个UI元素或3D对象处理。

4. 输入模式
游戏可以在不同的输入模式之间切换，如游戏模式和UI模式，影响输入的处理方式。

5. 事件传播
输入事件可以在UI层级或对象层级中传播，直到被处理或到达根节点。

6. 阻止传播
某些对象可以选择"消耗"输入事件，阻止它们继续传播。

7. 委托和事件
使用委托和事件系统来通知对象有关输入状态的变化。

以下是一个简化的示例，展示了如何在虚幻引擎中设置基本的输入路由：

```cpp:GameMode.cpp
void AMyGameMode::BeginPlay()
{
    Super::BeginPlay();

    // 获取玩家控制器
    APlayerController* PC = UGameplayStatics::GetPlayerController(this, 0);
    if (PC)
    {
        // 设置输入模式为游戏和UI
        FInputModeGameAndUI InputMode;
        InputMode.SetLockMouseToViewportBehavior(EMouseLockMode::DoNotLock);
        InputMode.SetHideCursorDuringCapture(false);
        PC->SetInputMode(InputMode);

        // 显示鼠标光标
        PC->bShowMouseCursor = true;
    }
}
```

```cpp:PlayerCharacter.cpp
void APlayerCharacter::SetupPlayerInputComponent(UInputComponent* PlayerInputComponent)
{
    Super::SetupPlayerInputComponent(PlayerInputComponent);

    // 绑定移动输入
    PlayerInputComponent->BindAxis("MoveForward", this, &APlayerCharacter::MoveForward);
    PlayerInputComponent->BindAxis("MoveRight", this, &APlayerCharacter::MoveRight);

    // 绑定鼠标输入
    PlayerInputComponent->BindAxis("Turn", this, &APlayerCharacter::AddControllerYawInput);
    PlayerInputComponent->BindAxis("LookUp", this, &APlayerCharacter::AddControllerPitchInput);

    // 绑定动作输入
    PlayerInputComponent->BindAction("Jump", IE_Pressed, this, &ACharacter::Jump);
    PlayerInputComponent->BindAction("Fire", IE_Pressed, this, &APlayerCharacter::OnFire);
}

void APlayerCharacter::OnFire()
{
    // 检查是否正在与UI交互
    APlayerController* PC = Cast<APlayerController>(Controller);
    if (PC && PC->WasInputKeyJustPressed(EKeys::LeftMouseButton))
    {
        FHitResult HitResult;
        PC->GetHitResultUnderCursor(ECC_Visibility, false, HitResult);

        // 如果没有点击UI元素，则执行射击逻辑
        if (!HitResult.bBlockingHit || !HitResult.GetActor()->IsA(UUserWidget::StaticClass()))
        {
            // 执行射击逻辑
            UE_LOG(LogTemp, Warning, TEXT("Fire!"));
        }
    }
}
```

在这个例子中：

1. `GameMode` 设置了输入模式，允许游戏和UI同时接收输入。

2. `PlayerCharacter` 设置了基本的移动和动作输入绑定。

3. `OnFire` 函数展示了如何处理可能与UI交互冲突的输入。它首先检查鼠标点击是否命中了UI元素，如果没有，才执行游戏逻辑。

这个系统允许开发者创建复杂的交互系统，同时处理3D世界和2D UI的输入。正确理解和使用输入路由可以帮助创建更加直观和响应迅速的用户体验。