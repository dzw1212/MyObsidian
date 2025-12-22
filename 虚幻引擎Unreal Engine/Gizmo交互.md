在虚幻引擎中，`FLegacyEdModeWidgetHelper` 是一个用于支持旧版编辑器模式（Legacy Editor Mode）的工具类，主要用于在编辑器模式下创建和管理自定义的 Gizmo（操纵器）。Gizmo 是用户在编辑器中与对象交互的重要工具，例如移动、旋转、缩放等操作。

`FLegacyEdModeWidgetHelper` 是一个辅助类，用于在旧版编辑器模式中管理 Gizmo 的显示和交互。它的主要功能包括：
- 管理 Gizmo 的显示状态。
- 处理 Gizmo 的输入事件（如鼠标点击、拖动等）。
- 与编辑器模式的其他部分进行交互。

---

# 创建自定义编辑器模式

首先，你需要创建一个自定义的编辑器模式。编辑器模式是用户在编辑器中选择的工具，例如“地形编辑模式”、“植被编辑模式”等。

- 继承 `FEdMode` 类，并实现必要的方法。
- 在 `Enter()` 和 `Exit()` 方法中初始化或清理 Gizmo。

```cpp
class FYourCustomEdMode : public FEdMode
{
public:
    virtual void Enter() override;
    virtual void Exit() override;
    virtual void Render(const FSceneView* View, FViewport* Viewport, FPrimitiveDrawInterface* PDI) override;
    virtual bool InputDelta(FEditorViewportClient* InViewportClient, FViewport* InViewport, FVector& InDrag, FRotator& InRot, FVector& InScale) override;

private:
    TSharedPtr<FLegacyEdModeWidgetHelper> WidgetHelper;
};
```

---

#### **2.2 初始化 `FLegacyEdModeWidgetHelper`**
在编辑器模式的 `Enter()` 方法中，初始化 `FLegacyEdModeWidgetHelper`，并设置 Gizmo 的类型和属性。

```cpp
void FYourCustomEdMode::Enter()
{
    FEdMode::Enter();

    // 初始化 WidgetHelper
    WidgetHelper = MakeShared<FLegacyEdModeWidgetHelper>();

    // 设置 Gizmo 类型（例如移动、旋转、缩放）
    WidgetHelper->SetWidgetMode(UE::Widget::WM_Translate);

    // 设置 Gizmo 的目标对象
    WidgetHelper->SetCurrentWidgetAxis(EAxisList::XYZ);
}
```

---

#### **2.3 渲染 Gizmo**
在 `Render()` 方法中，调用 `FLegacyEdModeWidgetHelper` 的渲染逻辑，将 Gizmo 绘制到视口中。

```cpp
void FYourCustomEdMode::Render(const FSceneView* View, FViewport* Viewport, FPrimitiveDrawInterface* PDI)
{
    FEdMode::Render(View, Viewport, PDI);

    if (WidgetHelper.IsValid())
    {
        // 渲染 Gizmo
        WidgetHelper->Render(View, Viewport, PDI);
    }
}
```

---

#### **2.4 处理 Gizmo 的输入事件**
在 `InputDelta()` 方法中，处理用户对 Gizmo 的交互（例如拖动 Gizmo 来移动对象）。

```cpp
bool FYourCustomEdMode::InputDelta(FEditorViewportClient* InViewportClient, FViewport* InViewport, FVector& InDrag, FRotator& InRot, FVector& InScale)
{
    if (WidgetHelper.IsValid())
    {
        // 处理 Gizmo 的输入事件
        return WidgetHelper->InputDelta(InViewportClient, InViewport, InDrag, InRot, InScale);
    }

    return false;
}
```

---

#### **2.5 注册自定义编辑器模式**
在模块启动时，注册你的自定义编辑器模式。

```cpp
void FYourCustomEdModeModule::StartupModule()
{
    FEditorModeRegistry::Get().RegisterMode<FYourCustomEdMode>(
        FYourCustomEdMode::EM_YourCustomEdModeId,
        LOCTEXT("YourCustomEdModeName", "Your Custom EdMode"),
        FSlateIcon(FEditorStyle::GetStyleSetName(), "LevelEditor.YourCustomEdMode"),
        true
    );
}
```

---

### **3. 自定义 Gizmo 的高级功能**
如果你需要更复杂的 Gizmo（例如自定义形状、颜色或交互逻辑），可以通过以下方式实现：

#### **3.1 自定义 Gizmo 的绘制**
- 继承 `FWidget` 类，并实现自定义的渲染逻辑。
- 在 `FLegacyEdModeWidgetHelper` 中使用自定义的 `FWidget`。

```cpp
class FYourCustomWidget : public FWidget
{
public:
    virtual void Render(const FSceneView* View, FPrimitiveDrawInterface* PDI) override;
};
```

---

#### **3.2 自定义 Gizmo 的交互**
- 在 `FWidget` 中重写 `InputDelta()` 方法，处理自定义的交互逻辑。

```cpp
virtual bool InputDelta(FEditorViewportClient* InViewportClient, FViewport* InViewport, FVector& InDrag, FRotator& InRot, FVector& InScale) override;
```

---

#### **3.3 动态调整 Gizmo 的属性**
- 通过 `FLegacyEdModeWidgetHelper` 的 `SetWidgetMode()` 和 `SetCurrentWidgetAxis()` 方法，动态调整 Gizmo 的类型和轴。

---

### **4. 示例代码**
以下是一个简单的自定义 Gizmo 示例：

```cpp
void FYourCustomEdMode::Enter()
{
    FEdMode::Enter();

    WidgetHelper = MakeShared<FLegacyEdModeWidgetHelper>();
    WidgetHelper->SetWidgetMode(UE::Widget::WM_Translate);
    WidgetHelper->SetCurrentWidgetAxis(EAxisList::XYZ);
}

void FYourCustomEdMode::Render(const FSceneView* View, FViewport* Viewport, FPrimitiveDrawInterface* PDI)
{
    FEdMode::Render(View, Viewport, PDI);

    if (WidgetHelper.IsValid())
    {
        WidgetHelper->Render(View, Viewport, PDI);
    }
}

bool FYourCustomEdMode::InputDelta(FEditorViewportClient* InViewportClient, FViewport* InViewport, FVector& InDrag, FRotator& InRot, FVector& InScale)
{
    if (WidgetHelper.IsValid())
    {
        return WidgetHelper->InputDelta(InViewportClient, InViewport, InDrag, InRot, InScale);
    }

    return false;
}
```

---

### **5. 总结**
- `FLegacyEdModeWidgetHelper` 是一个用于管理 Gizmo 的辅助类，适合在旧版编辑器模式中使用。
- 实现自定义 Gizmo 的关键步骤包括：创建自定义编辑器模式、初始化 `FLegacyEdModeWidgetHelper`、渲染 Gizmo 和处理输入事件。
- 如果需要更复杂的 Gizmo，可以继承 `FWidget` 并实现自定义的渲染和交互逻辑。

通过以上方法，你可以在虚幻引擎中实现自定义的 Gizmo，并集成到编辑器模式中。