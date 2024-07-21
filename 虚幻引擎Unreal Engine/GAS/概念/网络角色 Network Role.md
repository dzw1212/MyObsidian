在虚幻中，网络角色（Network Roles）用于区分对象在网络游戏中的不同权限和职责。

# 网络角色

1. **`ROLE_None`**：
   - **描述**：没有角色。
   - **权限**：无网络相关权限。
   - **使用场景**：通常不使用。

2. **`ROLE_SimulatedProxy`**：
   - **描述**：客户端上的模拟代理。
   - **权限**：只能接收服务器的更新，不能自主做出决定。
   - **使用场景**：用于非本地控制的对象，例如其他玩家的角色。

3. **`ROLE_AutonomousProxy`**：
   - **描述**：客户端上的自主代理。
   - **权限**：可以自主做出决定并将其发送到服务器。
   - **使用场景**：用于本地控制的对象，例如本地玩家的角色。

4. **`ROLE_Authority`**：
   - **描述**：服务器上的控制权。
   - **权限**：完全控制对象，可以接收和处理来自客户端的请求。
   - **使用场景**：用于服务器控制的对象，负责游戏逻辑和状态的管理。

# 权限和职责

- **`ROLE_None`**：没有网络相关权限，通常不使用。
- **`ROLE_SimulatedProxy`**：只能接收服务器的更新，不能自主做出决定。适用于非本地控制的对象。
- **`ROLE_AutonomousProxy`**：可以自主做出决定并将其发送到服务器。适用于本地控制的对象。
- **`ROLE_Authority`**：完全控制对象，可以接收和处理来自客户端的请求。适用于服务器控制的对象。

# 什么时候判断网络角色

在网络游戏开发中，判断网络角色非常重要，因为不同的角色有不同的权限和职责。以下是一些常见的场景：

1. **初始化和设置**：
   - 在对象初始化时，根据角色设置不同的属性和行为。
   - 例如，只有服务器（`ROLE_Authority`）可以生成和销毁对象。

2. **同步和更新**：
   - 在同步对象状态时，根据角色决定是否发送或接收更新。
   - 例如，客户端（`ROLE_SimulatedProxy`）只接收服务器的更新，而服务器（`ROLE_Authority`）负责发送更新。

3. **输入处理**：
   - 在处理玩家输入时，根据角色决定是否处理输入。
   - 例如，本地玩家（`ROLE_AutonomousProxy`）可以处理输入并将其发送到服务器。

4. **游戏逻辑**：
   - 在执行游戏逻辑时，根据角色决定是否执行某些操作。
   - 例如，只有服务器（`ROLE_Authority`）可以修改游戏状态和处理游戏规则。

# 示例代码

以下是一个示例，展示了如何在不同的场景中判断网络角色：

```cpp
void AMyActor::BeginPlay()
{
    Super::BeginPlay();

    if (GetLocalRole() == ROLE_Authority)
    {
        // 服务器上的初始化逻辑
        InitializeOnServer();
    }
    else if (GetLocalRole() == ROLE_AutonomousProxy)
    {
        // 本地玩家的初始化逻辑
        InitializeOnClient();
    }
    else if (GetLocalRole() == ROLE_SimulatedProxy)
    {
        // 其他玩家的初始化逻辑
        InitializeOnSimulatedClient();
    }
}

void AMyActor::Tick(float DeltaTime)
{
    Super::Tick(DeltaTime);

    if (GetLocalRole() == ROLE_Authority)
    {
        // 服务器上的更新逻辑
        UpdateOnServer(DeltaTime);
    }
    else if (GetLocalRole() == ROLE_AutonomousProxy)
    {
        // 本地玩家的更新逻辑
        UpdateOnClient(DeltaTime);
    }
}
```
