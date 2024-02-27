# Registry, Entity, Component

`entt::registry`默认表示`entt::basic_registry<uint32_t>`，basic_registry允许用户自定义Entity的类型（一般来说uint32_t已经足够）；

Entity由实体标识符（entity identifiers）表示；

## 创建与销毁实体

```c++
entt::registry registry;
auto entity1 = registry.create(); //创建一个无组件的实体，并返回实体标识符
std::cout << (uint32_t)entity1 << std::endl; //0

auto entity2 = registry.create();
std::cout << (uint32_t)entity2 << std::endl; //1

auto entity3 = registry.create();
std::cout << (uint32_t)entity3 << std::endl; //2

std::cout << registry.size() << std::endl; //3

registry.destroy(entity2); //销毁实体及其所有组件
std::cout << registry.size() << std::endl; //3

registry.release(entity2); //孤立实体，不销毁其组件
std::cout << registry.size() << std::endl; //3

registry.clear(); //销毁所有实体
std::cout << registry.size() << std::endl; //3
```

## 检查实体是否可用

```c++
bool bIsEntity1Valid = registry.valid(entity1);
std::cout << bIsEntity1Valid << std::endl;	//true
bool bIsEntity2Valid = registry.valid(entity2);
std::cout << bIsEntity2Valid << std::endl;  //false，被destroy
bool bIsEntity3Valid = registry.valid(entity3);
std::cout << bIsEntity3Valid << std::endl;  //true
```

## 获取实体的版本

```c++
auto entity1Version = registry.current(entity1); //0
std::cout << entity1Version << std::endl;
```

## 实体添加组件

```c++
struct Position
{
	float x, y, z;
};

struct Velocity
{
	float v;
};

//添加组件
registry.emplace<Position>(entity1, 0.f, 0.f, 0.f);
auto& velComponent = registry.emplace<Velocity>(entity1);
velComponent.v = 100.f;
```

## 判断是否拥有组件

```c++
bool all_have = registry.all_of<Position, Velocity>(entity1); //全都有
bool any_have = registry.any_of<Position, Velocity>(entity1); //至少有一个
```

## 更新组件

```c++
//如果实体已有对应组件，则更新
registry.patch<Position>(entity1, [](auto& pos) {
	pos.x = pos.y = pos.z = 1.f;
});
registry.replace<Velocity>(entity1, 200.f);

//如果不知道是新增还是更新，则使用emplace_or_replace
registry.emplace_or_replace<Position>(entity1, 2.f, 2.f, 2.f);
//或者先判断
if (registry.all_of<Velocity>(entity1))
	registry.replace<Velocity>(entity1, 500.f);
else
	registry.emplace<Velocity>(entity1, 500.f);
```

## 移除组件

```c++
registry.remove<Position>(entity1);
std::cout << registry.all_of<Position>(entity1) << std::endl; //false，组件Position被移除
```

```c++
//遍历所有实体，移除某种组件
registry.emplace_or_replace<Velocity>(entity1, 1.f);
registry.emplace_or_replace<Velocity>(entity3, 1.f);

registry.clear<Velocity>();
std::cout << registry.valid(entity1) << std::endl; //true
std::cout << registry.all_of<Velocity>(entity1) << std::endl; //false，组件被移除
std::cout << registry.valid(entity3) << std::endl; //true
std::cout << registry.all_of<Velocity>(entity3) << std::endl; //false，组件被移除
```

## 获取组件

```c++
//若不存在，会引发异常，可以使用try_get来捕获异常
if (registry.all_of<Position>(entity1))
{
	auto posComponent = registry.get<Position>(entity1); 
	std::cout << posComponent.x << std::endl; //10
	std::cout << posComponent.y << std::endl; //20
	std::cout << posComponent.z << std::endl; //30
}
```

## 监测组件的变化

### 使用接收器 sink

`on_construct`：新建组件时调用，如emplace；
`on_destroy`：移除时调用，如clear/remove；
`on_update`：更新时调用，如replace/patch/emplace_or_replace；

```c++
void Position_Construct(entt::registry& registry, entt::entity entity)
{
	std::cout << std::format("Entity{} position construct", (uint32_t)entity) << std::endl;
}
void Position_Update(entt::registry& registry, entt::entity entity)
{
	std::cout << std::format("Entity{} position update", (uint32_t)entity) << std::endl;
}

registry.on_construct<Position>().connect<&Position_Construct>();
registry.emplace<Position>(entity3, 1.f, 2.f, 3.f); //Entity2 position construct
registry.on_construct<Position>().disconnect<&Position_Construct>();

registry.on_update<Position>().connect<&Position_Update>();
registry.replace<Position>(entity1, 100.f, 200.f, 300.f); //Entity0 position update
registry.on_update<Position>().disconnect<&Position_Update>();
```

### 使用观察者 observer

observer可以连接一个指定的`registry`和`matcher`；
当registry中的实体的组件的变化被matcher匹配时，就会将对应的Entity塞入observer；

```c++
entt::observer observer;
observer.connect(registry, entt::collector.update<Position>()); //监测Position组件发生变化的实体

registry.replace<Position>(entity1, 100.f, 200.f, 300.f);
registry.replace<Position>(entity3, 1000.f, 2000.f, 3000.f);

std::cout << observer.size() << std::endl; //2
for (const auto entity : observer)
{
	std::cout << (uint32_t)entity << std::endl; //2 //0
}
observer.clear();
```

matcher由`entt::collector`生成，有两种类型的matcher：
1、观察者匹配；
`entt::collector.update<Component>()`
监测所有未被销毁的实体中对应组件的变化，捕获发生变化的实体；
2、分组匹配；
`entt::collector.group<Component1, Component2>(entt::exclude<Component3>)`
监测自上一次刷新之后的所有未被销毁的实体，捕获符合组要求的实体；

---
matcher的条件可以组合，比如下句：
```c++
entt::collector.update<sprite>().where<position>(entt::exclude<velocity>);
```
匹配要求即为sprite组件发生变化、拥有position组件、且没有velocity组件的实体；

## 排序

```c++
registry.sort<renderable>([](const auto &lhs, const auto &rhs) {
    return lhs.z < rhs.z;
});

registry.sort<renderable>([](const entt::entity lhs, const entt::entity rhs) {
    return entt::registry::entity(lhs) < entt::registry::entity(rhs);
});
```

## 基本功能

### 空实体

使用`entt::null`表示一个空的Entity；

```c++
registry.valid(entt::null); //false

//用作比较
const auto entity = registry.create();
const bool null = (entity == entt::null);
```

### 墓碑实体

使用`entt::tombstone`表示一个被删除的实体；

```c++
registry.valid(entt::tombstone); //false

//用作比较
const auto entity = registry.create();
const bool tombstone = (entity == entt::timbstone);
```

### to_entity

返回一个与组件实例相关联的实体；

```c++
Position pos;
const auto entity4 = entt::to_entity(registry, pos);
```

### invoke

`entt::invoke`允许将信号传递给组件的成员函数，为实体选择正确的组件并调用组件的方法，必要时传递参数；

```c++
registry.on_construct<clazz>().connect<entt::invoke<&clazz::func>>();
```

# View与Group

`view`是一种非侵入性工具，用于在**不影响其他功能或增加内存消耗**的情况下处理Entity和Component；
`group`是一种侵入性工具，可用于**提高性能**，但也需要为此付出代价；

view分为两种，**编译时view**与**运行时view**，前者需要提供组件列表，可以进行多种优化；后者是使用数字类型标识符构建的，速度比较慢；
创造和销毁view的开销并不大，因为不涉及任何类型的初始化；

group有三种不同的类型：
1. 完全拥有组 full-owning group；
2. 部分拥有组 partial-owning group；
3. 非拥有组 non-owning group；
组可以拥有一种或者多种组件类型，允许重新排列内存以加速迭代；

## view

单组件类型的view和多组件类型的view是不同的；

单组件类型的view拥有最佳的性能，允许获得返回元素的准确数量；
多组件类型的view返回的元素需要满足所有组件类型，只能获取返回元素的估计数量；

==不推荐存储view，因为view的构建成本非常低==；

```c++
// single type view
auto single = registry.view<position>();

// multi type view
auto multi = registry.view<position, velocity>();

// 使用过滤器
auto view = registry.view<position, velocity>(entt::exclude<renderable>);

// 使用for循环遍历
auto view = registry.view<position, velocity, renderable>();

for(auto entity: view) {
    // a component at a time ...
    auto &position = view.get<position>(entity);
    auto &velocity = view.get<velocity>(entity);

    // ... multiple components ...
    auto [pos, vel] = view.get<position, velocity>(entity);

    // ... all components at once
    auto [pos, vel, rend] = view.get(entity);

    // ...
}

// 使用each遍历
registry.view<position, velocity>().each([](auto entity, auto &pos, auto &vel) {
    // ...
});

//使用for + each，可以方便地提取出组件
for(auto &&[entity, pos, vel]: registry.view<position, velocity>().each())
{
    // ...
};

//使用view获取组件时，可以省略组件类型
auto view = registry.view<const renderable>();

for(auto entity: view) {
    auto [renderable] = view.get(entity);
    // ...
}
```

## view组合


view支持相互与、或从而形成新的序列；
```c++
auto view = registry.view<position>();
auto other = registry.view<velocity>();

auto pack = view | other;

for(auto entity: pack) {
    auto [pos, vel] = pack.get(entity); //组件顺序与组合顺序相同
    // ...
}
```

## 迭代顺序

通过use强行指定迭代顺序；
```c++
for(auto entity : registry.view<position, velocity>().use<position>()) {
    // ...
}
```

单组件类型的view支持反向迭代，但是多组件类型的view不支持；
```c++
auto view = registry.view<position>();

for(auto it = view.rbegin(), last = view.rend(); it != last; ++iter) {
    // ...
}
```

## 运行时view

