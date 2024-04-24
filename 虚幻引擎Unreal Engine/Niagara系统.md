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

