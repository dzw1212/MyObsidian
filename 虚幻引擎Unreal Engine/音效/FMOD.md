# 核心概念

## 1. Event (事件) —— 音频的“原子”单位

在 FMOD 中，Event 是最基本的可播放单元。你并不是直接在 UE 中播放一个 `.wav` 文件，而是播放一个 Event。Event 就像一个容器，里面不仅包含声音波形文件，还包含播放逻辑。

**核心特性：**
*   **逻辑封装：** 一个 Event 可以包含多个音频轨道、随机播放逻辑、循环逻辑等。例如，“脚步声”Event 可能包含 10 个不同的脚步样本，每次播放时随机选择一个。
*   **参数驱动 (Parameters)：** 这是 Event 最强大的功能。你可以定义参数（如 `Speed`、`Health`、`SurfaceType`），然后在 UE 中根据游戏状态实时更新这些参数。
    *   *例子：* 汽车引擎声音。根据 UE 传来的 `RPM`（转速）参数，Event 会在内部平滑地从低转速采样过渡到高转速采样。
*   **3D 设置：** 衰减（Attenuation）、空间化（Spatialization）都在 Event 层面设置。

**在 UE 中的应用：**
*   **资产类型：** `UFMODEvent`。
*   **使用方式：** 使用 `PlayEventAtLocation` 或将其附加到 Actor 上。
*   **关键点：** UE 只需要知道 Event 的名字（或引用）并负责触发它，至于声音具体怎么变（比如加混响、改变音高），都是在 FMOD Studio 内部做好的。

## 2. Bank (库) —— 资源打包与内存管理

Bank 是 FMOD 的文件打包格式。所有的 Event、音频样本数据（Waveforms）和结构数据都被构建（Build）成 `.bank` 文件。

**核心特性：**
*   **加载机制：** 音频引擎不能直接播放未加载进内存的数据。Bank 就是加载的单位。你不能单独加载一个 Event，必须加载包含该 Event 的整个 Bank。
*   **Master Bank：** 每个 FMOD 项目至少有一个 Master Bank，它包含全局设置、Bus 结构和 VCA 定义。还有一个 `Master.strings.bank`，包含所有路径的字符串名称（开发调试时必须加载，否则只能通过 ID 访问）。
*   **优化策略：** 通常会按场景或功能划分 Bank。
    *   *MainMenu.bank* (只在主菜单加载)
    *   *Level1.bank* (进入第一关时加载，离开时卸载)
    *   *Weapons.bank* (常驻内存)

**在 UE 中的应用：**
*   **自动加载：** 在 FMOD UE 插件设置中，通常可以将 Master Bank 设置为自动加载。
*   **手动加载：** 对于特定关卡的 Bank，可以使用 UE 的蓝图节点或 C++ 代码在关卡开始（BeginPlay）时加载，结束时卸载，以节省内存。

## 3. Bus (总线) —— 混音与信号流

Bus 是音频信号流动的通道，构成了混音的层级结构（Hierarchy）。所有的 Event 最终产生的音频信号都会流入某个 Bus，Bus 再汇入 Master Bus，最后输出给硬件。

**核心特性：**
*   **层级管理：** 类似于文件夹结构。
    *   Master Bus
        *   SFX Bus (音效)
            *   Weapons
            *   UI
        *   Music Bus (音乐)
        *   Voice Bus (语音)
*   **DSP 效果器：** 你可以在 Bus 上挂载效果器（混响、EQ、压缩）。如果在这个 Bus 上的所有声音都需要变闷（例如玩家潜入水中），只需在这个 Bus 上加一个低通滤波器即可。
*   **Snapshot (快照)：** Snapshot 本质上是一种特殊的 Bus 设置覆盖。比如“濒死状态”Snapshot 激活时，会压低 SFX Bus 的音量，提升 Heartbeat Bus 的音量。

**在 UE 中的应用：**
*   **资产类型：** `UFMODBus`。
*   **动态混音：** 在 UE 中很少直接操作 Bus，通常是通过激活 **Snapshot** 来改变 Bus 的状态。
*   **调试：** 在开发中，如果觉得某个类别的声音太大，是在 FMOD Studio 的 Bus 界面调整，而不是去改每一个 Event。

## 4. VCA (Voltage Controlled Amplifier) —— 逻辑分组控制

VCA 源自模拟调音台的概念（电压控制放大器）。在 FMOD 中，VCA 是一个**完全独立于 Bus 层级**的音量控制推子。

**Bus 与 VCA 的区别（重点）：**
*   **Bus** 是物理信号流。声音必须经过 Bus。
*   **VCA** 是一个遥控器。声音**不流经** VCA。VCA 只是同时控制多个 Bus 或 Event 的最终音量。
*   *比喻：* 假设你有两个 Bus（"玩家枪声" 和 "敌人枪声"），它们在 Bus 结构中可能分开很远。但如果你想做一个 "所有武器音量" 的设置选项，你可以把这两个 Bus 都指派给同一个 "Weapons VCA"。推拉 VCA，这两个 Bus 的音量都会变。

**核心特性：**
*   **解耦：** VCA 允许你跨越 Bus 的层级结构来控制一组声音的音量。
*   **无 DSP：** VCA 只能控制音量（Volume），不能加混响或 EQ。

**在 UE 中的应用：**
*   **资产类型：** `UFMODVCA`。
*   **设置菜单 (Options Menu)：** VCA 是实现游戏设置菜单中“主音量”、“音乐音量”、“音效音量”滑块的最佳方式。
    *   你不应该直接改 Bus 的音量，因为 Snapshot 可能会覆盖 Bus 的音量设置。
    *   VCA 的改变是叠加在 Bus 现有的混合状态之上的。
*   **蓝图实现：** `Set VCA Volume` 节点直接对应 UI 里的 Slider。

# Event参数传递

**Parameter（参数）** 是 FMOD 与 UE 交互的灵魂。如果没有参数，FMOD 就只是一个简单的播放器；有了参数，声音才能“活”起来，对游戏状态做出实时反应。
## 一、在 FMOD Studio 中定义参数

在 FMOD Studio 中，参数本质上是一个**变量**。你不是在时间轴（Timeline）上线性播放声音，而是让时间轴或音量/音高根据这个变量的值发生变化。

### 1. 参数的类型
当你创建一个 Parameter 时，有几种主要类型：

*   **Continuous (连续型):**
    *   最常用。是一个浮点数范围（例如：`0.0` 到 `100.0`）。
    *   *用途：* 速度（Speed）、转速（RPM）、健康值（Health）、距离。
*   **Discrete (离散型):**
    *   整数步进（例如：`0`, `1`, `2`, `3`）。
    *   *用途：* 武器等级、连击数、开关状态。
*   **Labeled (标签型) —— 强烈推荐:**
    *   本质是离散型，但给每个整数起了个名字（例如：`Surface` 参数包含 `Grass`, `Concrete`, `Water`）。
    *   *用途：* 地面材质、武器类型。**这对 UE 程序员非常友好，因为代码里不用写 `0` 或 `1`，而是直接对应标签。**
*   **Built-in (内置型):**
    *   不需要 UE 传递，FMOD 自动计算。
    *   *常见：* `Distance`（听者与声源距离）、`Elevation`（高度差）、`Event Cone Angle`（声源朝向）。
    *   *用途：* 自动做 3D 衰减曲线。

### 2. 参数如何工作？
*   **自动化 (Automation):** 你可以在参数轴上画曲线。例如，当 `RPM` 参数升高时，音高（Pitch）升高，且“低速引擎循环”的音量降低，“高速引擎循环”的音量升高。
*   **磁头移动:** 参数甚至可以控制播放磁头的位置。

## 二、在 UE 中传递参数 

在 UE 中，你主要通过 `UFMODAudioComponent` 或直接对 Event 实例进行操作来传递参数。

### 场景 1：持续性声音（例如：汽车引擎、环境风声）
这种声音通常挂载在 Actor 上，持续播放，参数需要每帧（Tick）更新。

**1. 蓝图 (Blueprint) 实现:**
假设你的 Actor 上有一个 `FMOD Audio Component` 组件，并且已经选好了 Event（比如 `CarEngine`）。

```cpp
// 假设 Header 中已经定义了 UFMODAudioComponent* EngineAudioComp;

void AMyCar::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);

    float CurrentRPM = GetVehicleRPM(); // 获取游戏逻辑数值

    // 核心代码：直接设置参数
    if (EngineAudioComp)
    {
        EngineAudioComp->SetParameter(FName("RPM"), CurrentRPM);
    }
}
```

### 场景 2：一次性声音（例如：脚步声、枪声）
这种声音通常是“即发即弃”（Fire and Forget），但发射的那一瞬间需要知道上下文（比如踩在什么材质上）。

```cpp
// 播放声音并获取实例句柄
FFMODEventInstance Instance = UFMODBlueprintStatics::PlayEventAtLocation(GetWorld(), FootstepEvent, Location, true);

// 立即设置参数 (假设 0=Wood, 1=Concrete)
Instance.SetParameter(FName("Surface"), 1.0f);
```


### 三、一个完整的实战例子 —— 脚步声 (Footsteps)

#### 1. FMOD Studio 设置
1.  创建一个 Event 叫 `SFX_Footstep`。
2.  创建一个 **Labeled Parameter** 叫 `Surface`。
    *   Value 0: "Grass"
    *   Value 1: "Wood"
    *   Value 2: "Water"
3.  在 Event 的时间轴上创建三个轨道（Track）。
    *   轨道 1 放草地采样，只在 Parameter 指向 "Grass" 时激活。
    *   轨道 2 放木头采样，只在 Parameter 指向 "Wood" 时激活。
    *   以此类推。

#### 2. UE 设置
1.  **物理材质 (Physical Material):** 在 UE 项目设置中定义物理材质：`PM_Grass`, `PM_Wood` 等，并赋予地面的材质球。
2.  **角色动画通知 (Anim Notify):** 在角色跑步动画的“脚落地”那一帧，添加一个 Notify，比如 `PlayFootstep`。
3.  **蓝图逻辑 (Anim Blueprint):**
    *   当 `PlayFootstep` 触发时：
    *   执行 **LineTraceByChannel**（向脚下打射线）。
    *   获取 `Hit Result` -> `Phys Mat`（物理材质）。
    *   **Switch on Physical Material**:
        *   如果是 `PM_Grass`: 设定局部变量 `SurfaceVal = 0.0`。
        *   如果是 `PM_Wood`: 设定局部变量 `SurfaceVal = 1.0`。
    *   调用 **PlayEventAtLocation** (播放 `SFX_Footstep`)。
    *   拿到返回值 Instance，调用 **Set Parameter**，Name 填 `"Surface"`，Value 填 `SurfaceVal`。
