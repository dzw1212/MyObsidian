# 类型

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203002823.png)

## 关卡蓝图

### 事件

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203003126.png)

## 角色蓝图

### 创建第一视角角色蓝图

仿照示例项目的第一人称角色蓝图做一个类似的角色蓝图；

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231207234153.png)

角色蓝图自带胶囊体组件（箭头+网格体）和角色移动组件；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231207235930.png)

#### 添加骨骼网格体

添加一个摄像机与骨骼网格体，并指定骨骼网格体资产；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231208001711.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231208001855.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231208001723.png)

给骨骼网格体添加一个动画蓝图；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231208002050.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231208002104.png)

在World Setting中，将新建的角色蓝图设为Default Pawn Class，在PIE中查看；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231208004025.png)


![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231208004004.png)


#### 添加移动与视角控制

[[增强输入 EnhancedInput]]

WASD移动：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210143717.png)

Space跳跃：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210143945.png)

鼠标移动视角：
默认情况下，摄像机是跟着胶囊体移动的，因此只能左右不能上下，需要启用摄像机跟随Pawn旋转；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210145403.png)


![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210145243.png)


## Actor蓝图

### 创建一把枪

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210150635.png)

#### 创建骨骼与碰撞体

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210150811.png)

为骨骼网格体指定资产；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210150838.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210151022.png)

#### 枪的刷新点逻辑

创建一个带有聚光灯的蓝图，作为枪在场景中的刷新点；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210153147.png)

在关卡蓝图中，于刷新点处创建枪；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210153316.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210153455.png)

#### 枪的拾取逻辑

在角色的胶囊体与枪的胶囊体发生碰撞后，会触发`ActorBeginOverlap`事件；在检测到Overlap后，将枪装备在手上，需要知道手的骨骼名称；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210155400.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210160458.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210160552.png)

#### 创建子弹

添加静态网格体组件与投射物移动组件；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210161610.png)

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210161704.png)

设置投射物初始速度、最大速度与重力系数；

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210161753.png)

#### 枪的开火逻辑

由于枪作为Actor而非Pawn，因此无法直接接受到输入事件，需要Player进行事件转发；
为此，就需要Player持有枪的变量，在拾取到枪时给变量赋值；

创建IA_testFire，其值类型为bool，触发机制为**按下**和**按住**；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210164410.png)

在`IMC`中与鼠标左键相关联；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210163835.png)

角色蓝图中创建枪的变量，并在拾取时赋值；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210164812.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210165632.png)


为枪添加自定义事件Fire；

![150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210165339.png)

当收到IA_testFire的输入事件后，调用枪的Fire事件；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210165445.png)

#### 子弹射出逻辑

在枪的skeleton中找到射出点骨骼名称，将其所在位置作为子弹的生成点；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210170555.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210171024.png)

#### 射击音效与动画

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210193252.png)


# 通用

## 变量

### 类型

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203003447.png)

#### 数组

创建与添加元素：
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231204235825.png)

或者直接在蓝图面板中：
![150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231205002804.png)

长度：
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231205000042.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231205000528.png)

获取元素（复制或引用）：
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231205000147.png)

查找元素：若找到，返回下标；找不到，返回-1；
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231205000452.png)

插入元素：指定Index；
![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231205001017.png)

移除元素（指定元素或指定Index）：
![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231205001115.png)

判断是否含有元素：
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231205001303.png)

遍历数组：
![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231205001942.png)


### 获取与设置

获取变量：按住Ctrl将变量拖入蓝图；
设置变量：按住Alt将变量拖入蓝图；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203003724.png)

### 分组

1. 创建分组

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203004702.png)

2. 选择分组

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203004624.png)

### 运算与比较

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203004901.png)

## 键盘与鼠标输入

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203013047.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203013124.png)


## 获取选中的物体

场景中选中一个物体后，回到蓝图中右键；

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231203003954.png)

## 控制流

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231204232853.png)

Flip Flop节点：单数次运行时执行A，复数次运行时执行B；
![130](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231204233443.png)

Gate节点：状态为Open时向下执行，状态为Close时不向下执行；可以通过Toggle切换状态；
![230](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231204233529.png)

Multi-Gate节点：默认沿着Out0、Out1、...依次往下执行；可选择复位或循环；Start Index为-1时表示默认，为0时表示从第一个输出节点开始；
![230](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231204233922.png)


## 函数

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210205441.png)

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210205523.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210210744.png)

### 纯函数

1. 函数内部的操作不会对外部产生影响（如修改全局变量的值）；
2. 相同的输入总会得到相同的输出；

纯函数与非纯函数的一个重要差别就是——**纯函数节点不会缓存结果**，当一个纯函数节点被多个节点引用时，其会被重复多次调用；因此，建议一个纯函数只被一个节点引用，如果需要多次引用，建议使用本地变量保存单次调用的结果；

### 局部变量

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210210315.png)

### 与自定义事件的区别

1. 函数有局部变量和返回值，自定义事件没有；
2. 函数中不能使用`delay`节点，自定义事件可以；

### 与宏的区别

1. 宏可以有多个输入、输出引脚；
2. 其他蓝图可以调用函数，但不能调用宏；

## 宏

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210213220.png)

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210213157.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210213243.png)

### 宏库

宏库是一个管理多个宏的文件，可以在所有蓝图中调用；

创建宏库；

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210222329.png)

可以添加多个宏；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210222425.png)

在其他蓝图中调用；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210222820.png)


## 时间轴

在指定的时间内，将值从初始值变化到目标值；

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210224047.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210224014.png)

## 插值

与时间轴节点实现完美配合；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210224634.png)

## 与关卡蓝图交互

蓝图类中可以调用关卡蓝图中定义的事件，通过`Execute Console Command`节点，并且事件名前必须加上`ce `；

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210232728.png)

