`GameInstance` 在虚幻引擎中地位非常特殊，可以用一句话概括它的核心作用：

**它是游戏中“唯一的永生者”，用来存储所有需要在“切换关卡”时依然保留的数据和逻辑。**

为了让你完全理解它的用处，我们需要先看一个残酷的事实：**在虚幻中，当你调用 `OpenLevel` 切换地图时，几乎所有的东西都会被销毁。**

*   你的角色（Character）？没了。
*   当前的规则（GameMode）？重置了。
*   所有的 UI（Widget）？清空了。
*   关卡蓝图里的变量？全部归零。

这时候，**GameInstance** 就闪亮登场了。

---

# GameInstance 的三大核心用途

## 1. 跨关卡传值（最常用的功能）

比如你做了一个 RPG 游戏：
*   玩家在 `Map_Forest`（森林关卡）里打怪，此时还剩 **50% 血量**，背包里有 **3 个苹果**。
*   玩家走到了传送门，切换到 `Map_Town`（城镇关卡）。

**如果没有 GameInstance：** 玩家进入城镇时，新的 Character 被生成，血量又是 100%，苹果也没了。

**使用 GameInstance：**
1.  在离开森林前，把“50%血”和“3个苹果”存入 GameInstance 的变量里。
2.  进入城镇后，新的 Character 生成，立刻读取 GameInstance 里的变量，把血量设为 50%，背包塞入 3 个苹果。

## 2. 全局游戏设置（Settings）

有些东西是不属于某个特定关卡的，而是属于整个应用程序的：
*   **音量大小**（BGM / 音效）
*   **画质设置**（分辨率 / 阴影质量）
*   **语言设置**
*   **玩家的皮肤/外观选择**

你不希望玩家每换一张图，音量就重置回 100% 吧？这些数据非常适合存在 GameInstance 里，因为只要游戏程序不关闭，GameInstance 就一直活着。

## 3. 网络联机与会话管理（Networking）

在多人游戏中，GameInstance 通常用来处理**不依赖于特定地图**的网络逻辑：
*   寻找服务器列表（Find Sessions）。
*   创建房间（Create Session）。
*   处理网络错误（比如掉线后回到主菜单）。

因为你在“主菜单”点击“加入游戏”并跳转到“游戏地图”的过程中，GameMode 会死掉，PlayerController 会重置，只有 GameInstance 能全程“拿着电话不挂断”，确保你顺利连入服务器。

---

# GameInstance 与其他类的区别（避坑指南）

很多新手容易混淆 GameInstance 和其他保存数据的地方：

| 比较对象 | GameInstance | 区别核心 |
| :--- | :--- | :--- |
| **vs GameMode** | **GameInstance 活得更久** | GameMode 只要换图就重置（适合只管当前这一局的规则，如比赛倒计时）；GameInstance 换图不重置。 |
| **vs SaveGame** | **GameInstance 是内存，SaveGame 是硬盘** | GameInstance 是“临时的全局数据”（关掉游戏程序就没了）；SaveGame 是“存档文件”（存到硬盘里，下次开机还有）。**通常流程是：从硬盘加载 SaveGame -> 存入 GameInstance -> 游戏中读写 GameInstance -> 存回 SaveGame。** |
| **vs PlayerState** | **GameInstance 不会自动同步** | PlayerState 是专门用于网络同步玩家状态的；**GameInstance 默认不进行网络复制（Replication）**。客户端有自己的 GI，服务器有自己的 GI，它们互不通气。 |
