在虚幻引擎（Unreal Engine, UE）中，**`FGuid`** 是一个非常基础且强大的结构体，代表 **Globally Unique Identifier（全局唯一标识符）**。

它在本质上是一个 **128位** 的数据结构（由 4 个 `int32` 组成：A, B, C, D），用于在几乎不可能重复的情况下标识任何东西。

以下是关于 `FGuid` 的详细用法，涵盖 C++、蓝图以及常见应用场景。

---

# 1. FGuid 的核心特点

*   **唯一性**：通过算法生成，重复的概率极低（可以说是天文数字分之一）。
*   **跨平台**：UE 自己实现的 GUID，不依赖 Windows 的 COM GUID。
*   **轻量化**：只是 4 个整数，便于存储、复制和网络传输。

---

# 2. C++ 中的用法

在使用前，通常需要包含头文件（Core 模块默认包含，但显式引用是个好习惯）：
```cpp
#include "Misc/Guid.h"
```

## A. 创建/生成 GUID
```cpp
// 1. 创建一个全新的随机 GUID
FGuid NewId = FGuid::NewGuid(); 

// 2. 创建一个空的/无效的 GUID (0-0-0-0)
FGuid EmptyId; // 默认构造函数初始化为0
// 或者
FGuid InvalidId = FGuid();
```

## B. 检查有效性
```cpp
if (NewId.IsValid()) 
{
    // ID 不是 0-0-0-0
}

// 也可以用来清除 ID
NewId.Invalidate(); // 重置为 0
```

## C. 转换为字符串 (ToString)
通常用于调试、打印日志或存入数据库/文件名。
```cpp
// 默认格式 (D): 00000000-0000-0000-0000-000000000000
FString Str = NewId.ToString(); 

// 指定格式
// EGuidFormats::Digits (无横杠): 00000000000000000000000000000000
FString CleanStr = NewId.ToString(EGuidFormats::Digits); 
```

## D. 从字符串解析 (Parse)
```cpp
FString GuidStr = "12345678-1234-1234-1234-1234567890AB";
FGuid LoadedId;
FGuid::Parse(GuidStr, LoadedId); // 将字符串转回 FGuid 结构体
```

## E. 比较
```cpp
if (IdA == IdB) { ... }
if (IdA != IdB) { ... }
```

## F. 确定性 GUID (Deterministic GUID)
如果你希望根据某个字符串（如物体名称）每次都生成相同的 GUID，而不是随机的：
```cpp
// 只要输入字符串一样，生成的 GUID 永远一样
FGuid DeterministicId = FGuid::NewDeterministicGuid(TEXT("MyUniqueObjectName"));
```

---

# 3. 蓝图 (Blueprint) 中的用法

在蓝图中，`FGuid` 通常被称为 **Guid**。

1.  **创建变量**：在变量类型中搜索 `Guid` 即可。
2.  **生成**：使用 **`New Guid`** 节点。每次执行该节点都会返回一个新的唯一 ID。
3.  **转换**：
    *   `Guid to String`: 转为字符串。
    *   `Parse String to Guid`: 字符串转 Guid。
4.  **比较**：使用 `Equal (Guid)` 或 `Not Equal (Guid)`。

---

# 4. 常见应用场景

这是 `FGuid` 最能发挥作用的地方：

## 场景一：保存/加载系统 (Save System)
这是最常用的场景。当你保存游戏时，你需要保存世界中的多个物体（例如：地上的5把枪）。
*   **问题**：当读取存档时，怎么知道存档里的“枪A”对应世界里的哪把枪？
*   **解决**：给每个需要保存的 Actor 添加一个 `UPROPERTY(SaveGame) FGuid InstanceId;`。
    *   在 `OnConstruction` 或 `BeginPlay` 中，如果这个 ID 无效，就生成一个新的 `NewGuid()`。
    *   保存时，将这个 ID 和数据存入文件。
    *   读取时，通过 ID 匹配，确保数据加载到正确的物体上。

## 场景二：运行时动态生成物品
在 RPG 游戏中，玩家背包里的每一件装备都需要唯一标识。
*   即使玩家有两把相同的“铁剑”，它们可能耐久度不同，或者附魔不同。
*   给每个物品实例生成一个 `FGuid`，用作 `TMap<FGuid, FItemData>` 的 Key。这样可以轻松区分完全相同的物品类型。

## 场景三：资源管理与软引用
虽然 UE 内部使用路径（Soft Object Path）来引用资源，但在某些自定义的资源管理系统或 DLC 系统中，开发者喜欢给自定义资源分配 GUID，以便在文件名改变或移动文件夹后，依然能通过查表找到对应资源。

## 场景四：多人游戏/网络同步
在非 UE 标准的网络架构（如 HTTP 请求后端数据库）中，客户端生成一个 Request ID (`FGuid`) 发送给服务器，服务器处理完后返回带有该 ID 的响应，客户端凭此 ID 知道是哪个请求完成了。

---

# 5. 进阶：作为 Map 的 Key

`FGuid` 非常适合用作 `TMap` 的 Key，因为 UE 已经为它实现了高效的哈希函数 (`GetTypeHash`)。

```cpp
// 定义
TMap<FGuid, FMyCharacterStats> CharacterStatsMap;

// 使用
FGuid HeroId = FGuid::NewGuid();
FMyCharacterStats Stats;
Stats.Health = 100;

CharacterStatsMap.Add(HeroId, Stats);

// 查找
if (CharacterStatsMap.Contains(HeroId))
{
    // ...
}
```
