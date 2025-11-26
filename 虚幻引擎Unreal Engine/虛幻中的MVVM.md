[[MVC与MVVM]]

# VM插件

启用插件`UMG ViewModel`：

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251123215513.png)

# 自定义ViewModel类

基于`MVVMViewModelBase`创建自己的类：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251123234522.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251123234826.png)

# 为Widget创建ViewModel

## ViewModel窗口

在想要创建VM的Widget中，打开VM窗口，选择VM类，创建；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251125012454.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251125012534.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251125012809.png)

## 创建类型
**创建类型决定了“谁”负责创建这个 ViewModel 对象，以及这个 ViewModel 的“生命周期”归谁管理**

### 1. Manual（手动）

*   **含义**： 
    Widget **不会**自动创建这个 ViewModel 的实例。在 Widget 初始化时，这个 ViewModel 的引用是空的（Null）。
*   **谁负责创建**：
    你需要通过 C++ 代码或蓝图逻辑（通常在 `OnInitialized` 或外部调用）手动将一个现有的 ViewModel 对象赋值给这个 Widget。
*   **适用场景**：
    *   **列表项（List View Entries）**：例如背包中的一个格子。格子本身不需要创建数据，它是从外部接收数据的。
    *   **依赖外部数据的 UI**：例如显示当前选中的敌人信息。敌人对象已经存在，UI 只是去“引用”它，而不是创建一个新的敌人。
    *   **C++ 主导的架构**：如果你的 C++ 逻辑已经管理了所有的 ViewModel，UI 只需要被动接收。

### 2. Create Instance（创建实例）

*   **含义**：
    每当这个 Widget 被创建时，它会**自动 New 一个**该 ViewModel 类的新实例。
*   **谁负责创建**：
    Widget 自身。Widget 拥有这个 ViewModel 的所有权。
*   **生命周期**：
    ViewModel 随 Widget 生，随 Widget 死。如果 Widget 被销毁，这个 ViewModel 也会被垃圾回收（GC）。
*   **适用场景**：
    *   **独立且临时的 UI 逻辑**：例如一个简单的计算器弹窗，或者仅用于当前界面的临时状态（如“当前页码”、“是否正在加载”等纯 UI 状态）。
    *   **不需要与其他界面共享数据的场景**。

### 3. Global ViewModel Collection（全局 ViewModel 集合）

*   **含义**：
    Widget 会去查找一个**全局唯一**的 ViewModel 实例。如果该实例不存在，通常会报错或根据设置自动创建并注册到全局。
*   **谁负责创建/管理**：
    MVVM Subsystem（MVVM 子系统）。这个 ViewModel 存在于 GameInstance 级别，跨越不同的关卡和 UI 依然存在。
*   **适用场景**：
    *   **游戏设置（Settings）**：音量、画质设置。这些数据在任何界面（主菜单、暂停菜单）都是共享且一致的。
    *   **玩家核心数据（持久化）**：如玩家的金币总数、角色等级（如果在整个游戏运行期间都需要频繁访问且不随关卡销毁）。
*   **注意**：你需要在项目设置或 C++ 中将该 ViewModel 类注册为 Global ViewModel。

### 4. Property Path（属性路径）

可以通过一个函数名来找到ViewModel，这个函数需要是一个返回MVVMViewModel的**const函数**；

在基类中创建这样的一个函数，确保继承自该基类的Widget都能找到：
![1200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251126235817.png)

然后将函数名填入：
![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251127000018.png)




### 5.  Resolver（解析器）

这是一种**抽象的、基于逻辑**的获取方式。Resolver 是一个实现了 `UMVVMViewmodelResolver` 接口的类（通常是 C++ 类，也可以在蓝图中扩展）。

*   **工作原理**：
    你不需要告诉系统“怎么走”，你只需要告诉系统“用哪个逻辑去找”。
    Resolver 内部封装了复杂的查找代码（比如遍历数组、查询子系统、或者根据 Tag 查找）。
*   **独立性**：
    它不依赖具体的变量名，只依赖类型或逻辑。
*   **特点**：
    *   **解耦**：UI 不需要知道 ViewModel 具体存在哪个变量里，只需要知道有一个 Resolver 能找到它。
    *   **可复用**：一个“当前武器解析器”可以用在任何需要显示武器的 UI 上，不用关心武器是存放在 Pawn 身上还是背包组件里。
    *   **更安全**：可以在 C++ 中处理空指针检查和复杂的异常情况。



