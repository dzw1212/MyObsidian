
# 机制
## 属性 Property

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229205741.png)

## 焦点 focus

继承自底层控件`Control`的控件拥有焦点相关的功能；

`focusPolicy`控制如何获取焦点：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241231014348.png)

`focusReason`描述是如何获取/失去焦点的：

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241231014547.png)

### 活动焦点 activeFocus

一般来说拥有焦点的控件只有一个，如果多个控件初始的`focus: true`，则后构造的控件会拥有焦点；

然而有一种控件名为`FocusScope`，其允许其子控件不独占焦点，会出现多个控件`focus`的情况，此时就需要活动焦点`activeFocus`来判断究竟哪个控件真正拥有焦点；

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241231020101.png)

## 信号 signal

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241231234634.png)

可以通过`Component`和`Loader`为同一个信号绑定不同的`Handler`：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101162318.png)

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101162339.png)

## 与C++交互

### QML可访问的全局变量

设置之后，所有的QML均可访问到该变量：

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101170436.png)

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101170643.png)
### QObject类

有一种特殊的对象`QtObject`，使用它可以将`property`设为私有：

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101171653.png)


![650](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101171626.png)

`QtObject`继承自`QObject`，可以自定义cpp类继承自`QObject`，实现类似`QtObject`的功能；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101171956.png)

对于新增的成员变量和函数，可以移动光标并按下`Alt+Enter`，会出现一些选项：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101172929.png)

IDE会自动生成一些函数：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101173327.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101173449.png)

### QML中注册cpp类

*ps：以下方法基于`QT 6.8`，之前的注册方法如`qmlRegisterType`已被废弃；*

前提是派生自`QObject`的类；

#### 可实例化的类

首先需要添加`QML_ELEMENT`宏；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101213653.png)

然后QML直接使用即可：

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101213754.png)

#### 不可实例化的类


### 信号与槽函数

1. QML发送信号，CPP接收并处理；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101214543.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101214506.png)

2. CPP发送信号，QML接收并处理；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101224009.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101223929.png)





## 鼠标区域 MouseArea

![550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229183151.png)

### 拖拽

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229181559.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/drag.gif)

### 鼠标事件传递

当`MouseArea`重叠时，需要考虑重叠区域的事件是否传递；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229182939.png)

## 状态 State与切换 Transition

`states`设置多种状态；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229150447.png)

`transitions`描述切换过程；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229155321.png)

## 组件Component与加载Loader

控件中会自带一个默认`Component`，一般通过以下方法使用：

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229160246.png)

此外还可以自定义`Component`，需要通过`Loader`进行加载；
==将控件封装进`Component`中，方便动态创建或销魂==；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229160948.png)

可以通过`Loader`对`Component`中的控件进行管理，比如修改属性、销毁等；

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229161704.png)


`Loader`中可以设置异步加载，并通过`Status`属性判断加载是否完成；

![380](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229162441.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229162255.png)

# 工具


## 动画 Animation
### 触发
#### 调用触发

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229152444.png)

#### 立即触发

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229153123.png)

### 动画组合 SequentialAnimation

按顺序依次播放；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229153945.png)


## 布局

### 列布局 Column

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101235639.png)

相关动画：

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250101235714.png)

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/col.gif)

### 行布局 Row

相比`Column`多了镜像的功能；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102003431.png)

### 流布局 Flow

会根据父控件的大小自动换行；

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102004041.png)

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102004104.png)

### 网格布局 Grid

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102004351.png)

![150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102004425.png)

### 获取子元素

可以通过`children`获取所有元素，需要注意的，如果有`Repeater`或`ListView`，则其子元素也会包含在内（对于`Repeater`，假如其中有3个子元素，则总共加上`Repeater`算4个；如果是`ListView`，则只算`ListView`1个）；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102025021.png)

但是并不是所有的子元素都有某个属性：

![150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102025107.png)

可以通过`instanceof + type`判断子元素是否是某种类型：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102025337.png)

## 定时器 Timer

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102032942.png)

控制：

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102033016.png)

# 控件


## 按钮 Button

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229185738.png)

`background`可以自定义按钮的背景显示，如果需要改变字体、绘制自定义大小的背景图片等，需要用到`contentItem`属性；

### 勾选框CheckBox

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229211032.png)

通过`nextCheckState`可以控制点击时状态的切换顺序，默认是`unchecked -> particalChecked -> checked`：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229213105.png)


### 按钮分组 ButtonGroup

通过`ButtonGroup`还可以实现分层功能，当`ButtonGroup`中的所有`Button`的`checked`状态都是`true`时，该`ButtonGroup`才是`checked`的；

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229212511.png)

![100](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229212407.png)

### 延迟按钮 DelayButton

持续按住之后会累计进度；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229213555.png)

### 点选框 RadioButton

天生排他；



![100](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229213736.png)


### 切换按钮 Switch

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229214528.png)

![150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229214547.png)

### 表格按钮 TabButton

常用于菜单栏，天生排他；

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229214845.png)

![150](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229214829.png)

### 圆角按钮 RoundButton

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229215116.png)

![50](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229215058.png)

## 文本 Text

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229221513.png)

### 文本样式 textFormat

支持富文本`Richtext`、`Markdown`、`CSS`等格式；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229221339.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229221324.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229221641.png)

### 换行方式 wrapMode

![1100](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229221829.png)

### 超链接

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229222229.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229222212.png)


## 图片 Image

### 加载资源

QML中使用图片，需要新建一个qrc文件，通过该文件来管理图片等资源；

![360](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229171650.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229171710.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229171730.png)

此外还需要修改CMakeList文件：

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229171832.png)

### 静态图片 Image

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229172128.png)

### 动态图片 AnimatedImage

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229172408.png)

控制播放速度与暂停：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241229173334.png)


## 弹窗 Popup

`Popup`在形式上类似`Rectangle`，但是有几个不同于普通`Rectangle`的特点：
1. 初始默认`visible`为`false`；
2. 对于`Popup`，即使其父控件的`visible`是`false`，只要自身的`visible`为`true`，就能显示出来；
3. `Popup`相对于其他普通控件的层级总是在最上层；`Popup`之间，即使是后绘制的也会在下面的层级，但是可以用`z`属性设置层级关系；
4. `Popup`失去焦点后会自动隐藏（默认`closePolicy`）；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230021909.png)


`open`显示弹窗：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230021231.png)

`close`关闭弹窗：

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230021312.png)

### 模态 Modal/非模态 Modeless遮罩 Overlay

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230022639.png)

## 重复器 Repeater

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230202716.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230202808.png)

## 获取子元素

`itemAt(index)`

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102024008.png)

## 列表 ListView

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230232004.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230232036.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230232112.png)

列表选项拖动行为`interactive`与`boundsBehavior`：

![550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241231013443.png)


## 获取子元素

`children`中还包含`contenItem`自身；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102030602.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250102030625.png)

## 组合框 ComboBox

最基础的形式：

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230232829.png)

![100](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230232806.png)

可编辑选项`editable`：

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230233926.png)

`Index/Value/Text`：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241230235418.png)

限制输入类型`validator`与`acceptableInput`：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241231010519.png)

## 自定义绘制

下拉按键 `indicator`：

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241231011459.png)

![200](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241231011446.png)


此外还可以自定义显示框的背景`background`、显示框的文本`contentItem`、下拉框中的每一项`delegate`、整个下拉框`popup`；

