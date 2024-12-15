`Subsystem` 是一种用于管理和组织游戏功能的模块化系统，提供了一种结构化的方式来实现和管理游戏的全局功能或服务，`Subsystem` 可以在不同的上下文中使用，比如游戏实例、世界、引擎等；

# 类型

1. **Engine Subsystem (`UEngineSubsystem`)：**
   - 这些子系统在引擎级别运行，通常用于管理与引擎生命周期相关的全局功能。
   - 例如，管理全局设置、引擎级别的服务等。

2. **Game Instance Subsystem (`UGameInstanceSubsystem`)：**
   - 这些子系统在游戏实例级别运行，适用于需要在整个游戏会话中保持状态的功能。
   - 例如，管理玩家数据、网络会话等。

3. **World Subsystem (`UWorldSubsystem`)：**
   - 这些子系统在世界级别运行，适用于与特定世界相关的功能。
   - 例如，管理关卡特定的逻辑、环境设置等。

4. **Local Player Subsystem (`ULocalPlayerSubsystem`)：**
   - 这些子系统在本地玩家级别运行，适用于与特定玩家相关的功能。
   - 例如，管理玩家输入、UI 状态等。

# 优点

- **模块化设计：** Subsystem 提供了一种模块化的方式来组织代码，使得功能更易于管理和扩展。
- **生命周期管理：** Subsystem 的生命周期由引擎管理，开发者可以专注于实现功能而不必担心初始化和销毁。
- **全局访问：** Subsystem 可以在其上下文中全局访问，方便在不同的游戏组件之间共享数据和功能。

# 如何使用 Subsystem

1. **创建 Subsystem 类：** 继承自适当的 Subsystem 基类（如 `UGameInstanceSubsystem`）。
2. **实现功能：** 在 Subsystem 类中实现所需的功能和逻辑。
3. **注册 Subsystem：** 确保 Subsystem 在项目中被正确注册，以便在运行时被引擎识别和管理。
4. **访问 Subsystem：** 使用引擎提供的 API 在需要的地方访问和使用 Subsystem。

