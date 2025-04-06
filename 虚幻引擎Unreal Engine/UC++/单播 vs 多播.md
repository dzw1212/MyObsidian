在虚幻引擎（Unreal Engine）的委托系统中，`DECLARE_MULTICAST_DELEGATE_OneParam` 和 `DECLARE_DELEGATE_OneParam` 的主要区别在于它们的功能和用途：

### 1. **单播 vs 多播**
   - **`DECLARE_DELEGATE_OneParam`**  
     这是一个**单播委托**，意味着它只能绑定**一个函数**（或一个可调用对象）。当委托被触发时，仅调用这一个函数。  
     - 适用于一对一的回调场景（例如，某个事件只需要通知一个特定的对象）。

   - **`DECLARE_MULTICAST_DELEGATE_OneParam`**  
     这是一个**多播委托**，允许绑定**多个函数**（通过`Add()`或`AddStatic()`等）。触发时，会按顺序调用所有绑定的函数。  
     - 适用于一对多的通知场景（例如，一个事件需要通知多个监听者）。

---

### 2. **返回值**
   - **单播委托** (`DECLARE_DELEGATE_OneParam`)  
     可以定义返回值类型（例如 `DECLARE_DELEGATE_RetVal_OneParam`），因为只有一个被调用函数。  
     - 多播委托**没有返回值**，因为多个函数的返回值无法统一处理。

---

### 3. **用法示例**
#### 单播委托（一对一）
```cpp
DECLARE_DELEGATE_OneParam(FMyDelegate, int32);

// 绑定函数
FMyDelegate MyDelegate;
MyDelegate.BindLambda([](int32 Value) { UE_LOG(LogTemp, Warning, TEXT("Value: %d"), Value); });

// 调用委托（仅触发一个函数）
MyDelegate.Execute(42);
```

#### 多播委托（一对多）
```cpp
DECLARE_MULTICAST_DELEGATE_OneParam(FMyMulticastDelegate, int32);

// 绑定多个函数
FMyMulticastDelegate MyMulticastDelegate;
MyMulticastDelegate.AddLambda([](int32 Value) { UE_LOG(LogTemp, Warning, TEXT("Lambda 1: %d"), Value); });
MyMulticastDelegate.AddLambda([](int32 Value) { UE_LOG(LogTemp, Warning, TEXT("Lambda 2: %d"), Value); });

// 广播委托（触发所有绑定的函数）
MyMulticastDelegate.Broadcast(42);
```

---

### 4. **其他注意事项**
   - **绑定方式**：单播委托用 `Bind()`，多播委托用 `Add()`（需手动管理移除，避免重复绑定）。
   - **执行方式**：单播委托用 `Execute()`，多播委托用 `Broadcast()`（或`Broadcast()`安全的`IsBound()`检查）。
   - **线程安全**：多播委托的 `Broadcast()` 是非线程安全的，需自行处理竞态条件。

---

### 总结
- 需要**单一回调** → 用 `DECLARE_DELEGATE_OneParam`（单播）。  
- 需要**通知多个对象** → 用 `DECLARE_MULTICAST_DELEGATE_OneParam`（多播）。  
- 多播委托更常见于事件系统（如 `Actor` 的 `OnDestroyed` 事件）。