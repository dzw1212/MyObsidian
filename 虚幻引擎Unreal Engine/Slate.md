Slate 是 Unreal Engine 的底层 UI 框架，使用 C++ 编写，它提供了构建用户界面的基础设施，允许开发者通过代码创建复杂的 UI 组件；

由于是用 C++ 编写的，Slate 提供了高性能和高度的灵活性。开发者可以完全控制 UI 的行为和外观；


# Slot槽位

Slot 是一种容器，它定义了子组件在父组件中的位置和布局属性，每个子组件通常都被放置在一个 Slot 中；

不同的父组件（如 `SVerticalBox`、`SHorizontalBox` 等）可能会定义不同类型的 Slot，以支持特定的布局需求；

在 Slate 中，Slot 通常通过链式调用的方式来配置。例如：

````cpp
SNew(SVerticalBox)
+ SVerticalBox::Slot()
    .HAlign(HAlign_Center)
    .VAlign(VAlign_Center)
    [ //括号表示Slot
        SNew(STextBlock)
        .Text(FText::FromString("Hello, World!"))
    ]
+ SVerticalBox::Slot()
    .AutoHeight()
    [
        SNew(SButton)
        .Text(FText::FromString("Click Me"))
    ]
````
