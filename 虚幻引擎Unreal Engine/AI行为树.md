AI行为树是一种用于控制角色行为的图形化工具，用于实现游戏中各个角色的智能行为；

# 模块

- 行为树：用于描述和控制角色决策与行为的图形化工具，是由节点组成的层次结构，每个节点代表不同的行为、条件、或者控制结构；

- 黑板：一种用于在运行时存储和共享变量数据的系统，通常与行为树一起使用，用于在不同节点之间传递信息与共享状态，可以存储角色的属性、状态等信息；

- 场景查询：用于回去有关游戏世界中场景元素的信息，以便角色做出合理的判断和选择；

# 行为树节点

## 复合节点 Composite

用于组织和控制其他节点执行顺序和逻辑；

### 选择器 Selector

按照优先级依次尝试执行子节点，一旦某个子节点返回成功，则整个选择器节点返回成功；如果子节点返回失败，则执行下一个子节点直到某个子节点成功或全部失败；

![150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104013036.png)

### 序列 Sequence

按顺序执行子节点，只有当全部子节点都返回成功时，序列节点才返回成功；只要有一个子节点返回失败，则整个序列节点返回失败；

![150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104013108.png)

### 并行 Parallel

左边部分为主任务，只能执行简单的任务；右边部分为并行任务，可以执行复杂任务，与主任务同时进行；

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240116032804.png)

## 任务节点 Task

表示具体的行为或动作，是行为树中的基本操作单元，每个任务节点对应一个具体行为，如移动/攻击/待机等；

### 自带任务

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104013807.png)

其中一些有趣的：

- `Finish with Result`：直接返回结果，一般用于调试；
- `Move Directlt Toward`：对于路径上的障碍物不选择绕路，而是冲撞过去；
- `Rotate to Face BBEntry`：旋转朝向某个点（需要该NPC的旋转由Controller控制而非运动控制）；
- `Run Behavior`：运行一个子行为树，需要确保子行为树的黑板与当前行为树一致；
- `Run Behavior Dynamic`：可以根据Tag选择是否运行该子行为树；
- `Wait Blackboard Time`：使用黑板中的一个float值作为Wait的事件数，相当于可以动态设置等待的时间；


### 新建任务

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104013854.png)

### 任务蓝图事件

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104015306.png)

对应BeginPlay，EndPlay，Tick事件；

### 返回值

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104015423.png)

### 与黑板交互

假如黑板中有一个名为"TargetPos"的类型为`Vector`的变量；

在任务蓝图中创建一个类型为`Blackboard Key Selector`的对应变量并公开；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104015654.png)

获取与设置值：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104015803.png)

详情面板中给该变量选择对应的黑板变量；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104015945.png)


![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104015919.png)


## 节点附加

### 装饰器

基于各种条件来决定其关联的节点是否应该被执行，可以在条件满足时立即打断或进入该节点；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107220208.png)

- `Blackboard`：使用单个黑板键作为判断依据；
- `Composite`：使用多个黑板键作为判断依据，可以使用and/or/xor之类的逻辑组合；
- `Compare BBEntries`：比较两个黑板键的大小作为判断依据；
- `Conditional Loop/Loop`：如果是true则循环执行该节点（几次），如果是false则只执行一次；
- `Cone Check/Keep in Cone`：以目标是否在锥体范围内为判断依据；
- `Dose Path Exist`：两个目标之间是否有导航路径；
- `Force Success`：强制该节点返回成功，一般用于调试；
- `Is at Location`：是否在/不在某个点的附近；
- `Is BBEntry Of Class`：某个黑板键是否属于某个类；
- `Time Limit`：限时，如果该节点的执行时间超时则退出该节点；



以`Blackboard`类型的装饰器为例；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240106234757.png)

`Notify Observer`用于控制节点执行状态的通知时机：
- `On Result Change`: 只有当节点结果发生变化时才通知；
```cpp
bool isInSight = true;
// 只在以下情况触发：
isInSight = false; // 触发，因为从true变为false
isInSight = false; // 不触发，因为false没有变化
isInSight = true;  // 触发，因为从false变为true
```
- `On Value Change`: 当被观察的值发生改变时通知；
```cpp
float health = 100.0f;
// 在以下情况触发：
health = 90.0f;  // 触发，因为值从100变为90
health = 80.0f;  // 触发，因为值从90变为80
health = 80.0f;  // 不触发，因为值没有变化
```


`Observer aborts`定义了当装饰器的条件发生变化时，行为树应该如何响应：
- `None`: 条件的变化不会导致当前运行的行为被中断；
- `Self`: 如果这个装饰器监视的条件发生了变化，只有这个装饰器所装饰的任务会被中断；
- `Lower Priority:` 如果这个装饰器监视的条件发生了变化，它将中断在行为树中优先级低于这个装饰器节点的所有任务；
- `Both`: 如果这个装饰器监视的条件发生了变化，它将中断这个装饰器所装饰的任务以及所有优先级低于这个装饰器节点的任务；


比如下面这两个Sequence，左边的中断条件即为变量`FinUser`未设值和变量`LastSeePos`未设值，当变量`FindUser`或`LastSeePos`被清空或设为false时就会进入左边的Sequence；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107210859.png)

### 服务

装饰器用于控制节点的执行流程（是否执行），而服务用于**在节点执行时提供周期性的功能或数据更新**；

比如下面创建一个设置最大行走速度的Service，并附加在节点上；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107212838.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107213258.png)

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107213331.png)

# AI控制器

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231231210423.png)

给角色蓝图设置AIController类；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231231210456.png)

在AIControll中运行行为树：

![550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104020932.png)


# 示例：AI范围内随机移动

## 自定义任务节点

自定义`FindRandomTarget`任务节点实现如下：

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104020429.png)

## 黑板实现变量交互

黑板中创建变量用于任务节点交互：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104020527.png)

## 行为树逻辑


![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240104020315.png)

注意：如果想要AI Actor平滑地转向，需要开启Actor中`CharacterMovementComponent`的`Use Controller Desired Retation`：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250311010453.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250311011117.png)


# EQS系统

`EQS`全称为环境查询系统（`Environmental Query System`），是虚幻引擎中用于AI决策支持的一个功能强大的工具；它允许开发者创建复杂的查询来评估游戏环境，并根据这些查询的结果来指导AI的行为；

使用`EQS`，开发者可以询问游戏世界中的各种条件，比如：
- 寻找最近的弹药补给；
- 确定哪个敌人最容易攻击；
- 寻找用于掩护的最佳位置；

`EQS`通过定义一系列的测试来工作，每个测试都会对环境中的一系列点进行评分。这些点可以是实际的游戏世界中的位置，也可以是潜在的目标对象。每个测试都会根据特定的标准来评分，比如距离、是否有遮挡、是否在视线中等。然后，`EQS`会综合所有测试的结果，选择得分最高的点作为查询结果；

这个系统非常适合实现复杂的AI决策逻辑，因为它可以考虑多个因素并生成一个最优解。EQS通常与行为树一起使用，行为树节点可以触发EQS查询，并根据查询结果来选择不同的行动路径。

## 创建EQS

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107224306.png)

还需要一个EQS测试员`EQS test pawn`；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107224518.png)

给EQS测试员指定网格体和EQS模板：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107224938.png)


## EQS查询模板

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107224344.png)

- `Actors Of Class`

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107225457.png)

选中EQS测试员其周围范围内的Actor会高亮；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107225659.png)

- `Points:Circle`

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107225952.png)

- `Points:Cone`

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107230031.png)

- `Points:Donut`

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107230327.png)


- `Points:Grid`

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107230240.png)

## 运行EQS

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107230949.png)

使用创建的EQS模板，并将查询到的目标与黑板进行交互；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107230918.png)


## 更改EQS检测员

需要使用`EnvQueryContext`；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107233051.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107231941.png)

将所有类型为BP_NPC的Actor作为目标Actor；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107233251.png)

在EQS中使用创建的EQContext；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107233759.png)

可以看到，检测的范围变成了以BP_NPC为中心，而不是EQS检测员；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107234042.png)

## 给检测球添加测试

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240107235356.png)

以距离测试为例，测试每个检测点与BP_NPC之间的3D距离；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240108000222.png)

### 测试目的

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240108000029.png)

#### 过滤

设置距离为100-300之间的会被过滤掉；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240108000614.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240108000702.png)

#### 得分

设置线性距离越远分数越高；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240108000806.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240108000831.png)

### 使用最佳测试点

选择得分最高的测试点，将其位置设置给TargetPos；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240108010720.png)

## EQS与黑板的交互

比如，将Grid检测中的GridSize绑定为参数；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240116032104.png)

然后将该参数与一个黑板键相关联；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240116032315.png)



## 自定义生成器

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240110000930.png)

`Class Default`详情面板中选择生成物类型，这里使用Point；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240110001204.png)

比如说，在Z轴方向垂直绘制检测点：

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240110004959.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240110005041.png)

效果：

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240110005112.png)

