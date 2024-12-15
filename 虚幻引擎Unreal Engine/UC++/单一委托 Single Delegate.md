单一委托是最简单的委托类型，它允许绑定一个单一的函数；当委托被调用时，只有这个绑定的函数会被执行；

适用于需要简单回调机制的场景，比如按钮点击事件、简单的状态变化通知等；

# 声明

```cpp
DECLARE_DELEGATE(DelegateName);

DECLARE_DELEGATE_OneParam(DelegateName, ParamType); //带参数

DECLARE_DELEGATE_RetVal(ReturnType, DelegateName); //限制返回值类型
```

# 定义

```cpp
class MyClass
{
public:
    FSimpleDelegate OnActionCompleted;
};
```

# 绑定

```cpp
void MyFunction()
{
    UE_LOG(LogTemp, Warning, TEXT("Action Completed!"));
}

void ExampleUsage()
{
    MyClass MyObject;
    MyObject.OnActionCompleted.BindStatic(&MyFunction);
}
```

# 调用

```cpp
void PerformAction()
{
	// 执行某些操作
	// ...

	// 操作完成后调用委托
	if (OnActionCompleted.IsBound())
	{
		OnActionCompleted.Execute();
	}
}
```