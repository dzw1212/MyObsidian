`EnTT` 是一个用 C++ 编写的快速且轻量级的实体组件系统（`ECS`）库。它旨在提供高性能和灵活性，使得开发者能够以数据驱动的方式构建复杂且高效的应用程序，特别是在游戏开发和系统模拟领域。EnTT 的设计哲学是尽可能地保持简单和最小，同时提供强大的功能集；

EnTT 的主要功能包括：
- 实体管理：EnTT 提供了一个高效的实体管理系统，允许快速创建、销毁和管理实体（`Entities`）。实体在 EnTT 中通常表示为简单的ID；

- 组件存储：组件（`Components`）是附加到实体上的数据结构，用于存储状态（如位置、速度、健康值等）。EnTT 提供了灵活的组件存储系统，支持快速访问和修改组件数据；

- 系统执行：系统（`systems`）是执行逻辑的函数或对象，它们处理具有特定组件集的实体。EnTT 支持创建和执行这些系统，以实现游戏或应用程序的核心逻辑；

- 视图和迭代：EnTT 提供了强大的视图（`views`）功能，允许开发者高效地迭代拥有特定组件集的实体集合。这对于编写性能敏感的逻辑非常有用；

- 事件系统：EnTT 包含一个事件系统，允许实体和系统之间的通信和事件传递。这有助于实现松耦合的架构；

- 标签和分组：支持为实体添加标签和分组，以便更灵活地组织和查询实体；

- 注册表快照：提供了创建和恢复注册表状态的快照功能，这对于实现游戏保存和加载等功能非常有用；

- 跨平台：EnTT 是跨平台的，支持多种编译器和平台，使其适用于各种类型的项目；

- 头文件仅库：EnTT 是一个头文件仅库，这意味着它不需要预编译；你只需要包含相关的头文件即可使用；

---

[[ECS是什么]]

假如要实现2D空间射击游戏系统，使用EnTT可以这么做：

1. 创建组件；
	定义几个只包含数据的结构体，作为组件；
```cpp
struct PositionComponent {
    float x, y;
};

struct HealthComponent {
    int health;
};

struct VelocityComponent {
    float vx, vy;
};
```

2. 创建实体，并添加相应的组件；
	EnTT中使用注册表（`registry`）来管理实体和组件；
```cpp
#include <entt/entt.hpp>

int main() {
    entt::registry registry;

    // 创建玩家实体并附加组件
    auto player = registry.create();
    registry.emplace<PositionComponent>(player, 0.0f, 0.0f);
    registry.emplace<HealthComponent>(player, 100);
    registry.emplace<VelocityComponent>(player, 0.0f, 0.0f);

    // 创建敌人实体并附加组件
    auto enemy = registry.create();
    registry.emplace<PositionComponent>(enemy, 10.0f, 10.0f);
    registry.emplace<HealthComponent>(enemy, 50);

    // 创建子弹实体并附加组件
    auto bullet = registry.create();
    registry.emplace<PositionComponent>(bullet, 5.0f, 5.0f);
    registry.emplace<VelocityComponent>(bullet, 1.0f, 1.0f);

    // 更多逻辑...
}
```

3. 创建处理组件中数据的系统；
	系统是执行逻辑的函数或对象，它们处理具有特定组件的实体；
```cpp
void MovementSystem(entt::registry& registry) {
    auto view = registry.view<PositionComponent, VelocityComponent>();
    for (auto entity : view) {
        auto& position = view.get<PositionComponent>(entity);
        auto& velocity = view.get<VelocityComponent>(entity);

        position.x += velocity.vx;
        position.y += velocity.vy;
    }
}

void RenderSystem(entt::registry& registry) {
    auto view = registry.view<PositionComponent>();
    for (auto entity : view) {
        auto& position = view.get<PositionComponent>(entity);
        // 假设的渲染函数
        RenderEntity(position.x, position.y);
    }
}
```

4. 在游戏循环中运行系统；
```cpp
int main() {
    entt::registry registry;
    // 创建实体和组件...

    while (gameIsRunning) {
        MovementSystem(registry);
        RenderSystem(registry);

        // 处理其他系统和逻辑...
    }
}
```

