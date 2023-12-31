虚幻示意图形UI系统（`Unreal Motion Graphic UI System`，`UMG`），用于实现虚幻引擎中的Hud与UI功能；

# 创建Widget Blueprint

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210235554.png)

`Widget Blueprint`拥有自己的界面，中心处为面板区，左上为控件，可以拖拽进面板，右边为详情面板；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210235726.png)

# 锚点 Anchors

锚点用于适配UI控件的相对位置，分为三类：
1. 九点适配：随着画布尺寸改变，控件左上角与对应点的距离不变，控件的宽高比例不变；
2. 单轴适配：随着画布尺寸改变，控件与画布左侧或者上侧的距离不变，控件的宽高比例随之改变；
3. 父节点尺寸适配：随着父控件的尺寸改变而改变；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231211003929.png)

# 渲染UI到视口

调用`Create Widget`节点，并选择要创建的控件蓝图，并添加到当前视口；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216174057.png)


# 修改控件的值

1. 将控件提升为变量，然后修改其值；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216210818.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216212457.png)


2. 将控件的值与某个变量进行绑定，在修改该变量的同时控件的值也会随之改变；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216212609.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216212808.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216212822.png)

# 自定义控件

可以引用其他的`Widget Blueprint`；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216225904.png)

引用过来的控件无法之间进行编辑，也无法直接添加子控件；如果需要添加子控件，需要给原来的控件蓝图上加一个命名插槽`Named Slot`；通过命名插槽，可以给引用的控件添加子控件；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216230804.png)

# UI动画

## 制作

创建动画，并为指定控件创建轨道；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217154734.png)

添加动画关键帧；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217154903.png)

## 播放

创建好的动画可以在事件图表的`Animations`栏中找到；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217155143.png)


# 控件

## 富文本块 RichTextBlock

首先创建一个数据表格，类型选择`RichTextStyleRow`；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216215810.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216215834.png)

创建文本样式，第一行样式的名称必须是`Default`；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216222609.png)

指定文本样式集，并使用`css`标记指定的文本块；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216223322.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216223336.png)

## 边界 Border

边界控件中只能存在一个子控件，其子空间不能设置锚点和位置，取而代之的是填充（距离边缘的距离）和对齐；

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216231446.png)

其主要作用就是给控件加上一层轮廓；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231216231516.png)

# 面板

## 画布 Canvas

画布的主要作用为调整控件的位置，超出画布之外的控件不渲染；
画布本身没有位置，只能通过设置分辨率的方式调整大小；

创建画布：

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231211000211.png)

缩放以设置大小，此处设置为1920x1080：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231211000246.png)

## 覆层 Overlay

Overlay相当于经过拓展的Border，同样是只可以设置填充和对齐，不同的是Overlay可以容纳多个子控件；

此外，Overlay中的子控件存在覆盖关系，越靠下的控件的层级越高；

## 包裹框 WrapBox

包裹框会对其子控件进行排列，如果一行放不下还会换行；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217141435.png)

`Explicit Wrap Size`如果启用，会忽视包裹框的大小，使用`Wrap Size`的值判断是否换行；

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217141629.png)

## 控件切换器 Widget Switcher

控件切换器中可以容纳多个子控件，但是这些子控件必须是同一类型；

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217142827.png)

子控件默认会填充整个切换器，切换器可以通过索引控制当前显示哪个子控件；

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217142943.png)

## 统一网格面板 Uniform Grid Panel

所以子控件统一大小，均匀排列，通过箭头调整子控件的位置；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217144502.png)

## 网格面板 Grid Panel

允许子控件大小不同；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217151012.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217150951.png)

## 尺寸框 Size Box

只能有一个子控件，用来限制子控件的大小；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217161548.png)

## 滚动框 Scroll Box

相当于Listbox；

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231217162246.png)
