`Niagara`是一个基于节点的可视化特效编辑器，可以用来创建复杂和高性能的粒子系统，以及其他类型的视觉效果；

核心组件有：
1. 系统`System`：系统是`Niagara`中最顶层的组件，负责管理整个粒子系统的生命周期，包括粒子的生命周期、发射速度、数量等；
2. 发射器`Emitter`：发射器是粒子系统的发射源，可以连续不断得向空中发射粒子；每个发射器都有自己的粒子样式和发射速率，可以根据需要调整；
3. 模块`Module`：模块可以用来修改粒子的属性、行为和外观等；
4. 参数`Parameter`：参数用于控制粒子系统的各种属性，比如粒子数量、颜色、速度等；


# 创建Niagara

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240121221347.png)

创建一个全新的空的Niagara；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240121221457.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240121221516.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240121224007.png)

# 阶段 Stage

类似`Emitter Spawn`，`Emitter Update`，`Render`之类的称为`Stage`，在执行时会按顺序依次从上往下执行内部的`Module`（`Render`除外，顺序不一定从上到下）；

- `Emitter Spawn`：发射器生成时执行一次；
- `Emitter Update`：发射器每帧执行一次；
- `Particle Spawn`：每个粒子生成时执行一次；
- `Particle Update`：每个粒子每帧执行一次；
- `Render`：用于定义粒子的显示，可以有0个或者多个渲染器；

# 模块 Module

类似于函数，比如`Particle Spawn`下的`Initialize Particle`，模块在`Stage`下一般从上往下执行；

## Emitter State(必需)

默认`system`配置，也可以选择自定义的`self`配置；

主要包含循环行为`Loop Behavior`（永久`Infinite`，单次`Once`，多次`Multi`），循环周期`Loop Duration`等；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729020525.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729020621.png)

## Spawn Rate

用于持续生成粒子；

生成速率`Spawn Rate`：每秒钟生成多少个粒子；
生成概率`Spawn Probability`：有多大的概率生成粒子；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729020910.png)

## Spawn Burst Instantaneous

用于在特定时间点瞬间发射大量粒子；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729022419.png)


## Spawn Per Unit

根据发射器的移动距离来生成粒子，粒子的生成数量与发射器的移动距离成正比；
对于创建轨迹效果或基于路径的粒子效果非常有用；

最大速度限制`Max Movement Threshold`；
最小速度限制`Movement Tolerance`；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729023130.png)

## Initialize Particle(必需)

用于粒子的初始化；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729023541.png)

### Point属性

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729025820.png)

生存时间`Lofetime Mode`：可以选择固定的，也可以是随机的；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729025928.png)

颜色`Color Mode`：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729030018.png)

### Sprite属性

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729030654.png)

## Add Velocity

用于控制粒子的运动方向和速度；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729031317.png)

速度模式`Velocity Mode`:

1. 线性`Linear`：

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/ani6.gif)


2. 从一个点处扩散`From Point`：

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/ani5.gif)

3. 锥形`In Cone`：

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/ani7.gif)

## Shape Location

用于控制粒子在特定形状内生成的位置，从而实现各种复杂的粒子效果；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240729040122.png)


## Scale Color

用于调整粒子的颜色，这个模块允许根据粒子的生命周期或其他属性来动态地改变其颜色；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240802001527.png)

## Gravity Force

用来模拟重力效果，会对粒子施加一个向下的力，使得粒子看起来像是受到了重力的影响；

![550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240802001712.png)

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/ani8.gif)

## Drag

用于模拟空气阻力或其他形式的阻力对粒子的影响；
这个模块会减慢粒子的速度，使其逐渐停止或减速，通常用于创建更真实的粒子运动效果，例如烟雾、尘埃或其他需要模拟空气阻力的效果；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240802005100.png)

## Curl Noisy Force

用于生成复杂、自然的流体运动效果的力场模块；
通过使用`Curl Noise`算法来模拟涡流和湍流效果，使粒子系统看起来更加逼真和自然；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240802005336.png)

如果未使用`Curl Noisy Force`模块，则下面每个粒子应该直线运动；

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/ani9.gif)

