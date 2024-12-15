Common UI是一个UI插件，是Slate/UMG框架的扩展，其提供了一个丰富的工具箱，可用于创建丰富的、多层级的并且支持跨平台的用户界面；

# Common UI核心理念

- [[输入路由 Input Routing]]：确保指定时间内只有特定的UI空间能够收到来自用户鼠标或手柄的输入；

	详情参考`CommonGameViewportClient.h`与`CommonUIActionRouterBase.h`；

- 节点 Nodes：将UI控件转化为树状分布的节点，形成基于视觉和逻辑上的层级关系；

	每次刷新，Common UI或找到最顶层的树，将输入发送给这个树的根节点控件，根节点再将这个输入转发给第一个可以接收此输入的节点控件，依次转发直到被某个控件block；

	详情参考`UIActionRouterTypes.h`；

![350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241019172309.png)

- 可激活控件 Activatable Widgets：不是所有的节点控件都能接收输入，只有特定的控件才会被转化为需要处理输入的节点，这些被称为可激活控件；

	可以通过开关激活或关闭；
	可以将输入转发给同一棵树上的已激活控件；
	可以设置条件在合适的时候自动关闭；

	详情参考`CommonActivatableWidget.h`；

# 项目设置Common UI

## Step0、启用Common UI插件

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021222028.png)


## Step1、设置视口类，使其支持输入路由

视口（Viewport） 是Common UI全部输入路由的基础。当Common UI捕获输入时，会将其发送至视口，然后视口会将其发送至顶部的节点；

通用设置中，设置客户端视口类为`CommonGameViewportClient`，如果想用自定义的视口类，需要继承自该通用视口类；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021223546.png)


## Step2、创建输入动作数据表，将控制器按键映射到UI中

PS：Common UI的输入动作数据表与项目设置中的输入动作或者高级输入系统无关，它们仅用于管理UI输入；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021223845.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021224217.png)

输入动作的参数：
	显示名称`Display Name`：输入动作的名称，如果有导航栏，会显示在导航栏中；
	按住显示名称`Hold Display Name`：需要用户按住按钮时的输入动作的名称；
	导航栏优先级`Nav Bar Priority`：在导航栏中从左到右排列动作时的优先级；
	键盘输入类型信息`Keyboard Input Type Info`；
	默认手柄输入类型信息`Default Gamepad Input Type Info`；
	游戏手柄输入覆盖`Gamepad Input Overrides`：使用特定游戏手柄时执行该动作的按键；
	触控输入类型信息`Touch Input Type Info`；

比如此处定义两个输入动作，一个鼠标点击，一个键盘Esc：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021231149.png)


## Step3、默认导航动作设置

Common UI使用`CommonUIInputData`来定义所有平台通用的**点击**、**返回**输入动作；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021230434.png)

使用刚才定义的两个输入动作：

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021231301.png)

装载该输入数据用于默认导航：

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021231624.png)

## Step4、控制器数据绑定

控制器数据资产`ControllerDataAsset`将按键动作与UI元素关联，每个控制器数据资产都与一个输入类型、游戏手柄或平台关联，Common UI会使用该信息来根据当前平台和输入类型自动使用特定平台的正确UI元素；

初次之外，还可以根据用户的输入来找到正确的输入类型并在运行时切换UI元素（比如使用键鼠时突然切换为手柄）；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021234124.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021234428.png)

控制器数据的参数：
	输入类型`Input Type`：有键鼠、手柄、触摸板；
	手柄名称`Gamepad Name`：默认为`Generic`；
	输入刷数据映射`Input Brush Data Map`：从按键到UI元素和图标的映射；
	输入刷按键组`Input Brush Key Sets`：用于多个按键映射到单个UI元素；

将控制器数据资产添加到关联的平台上：

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20241021235735.png)

## Step5、Common UI控件库与控件样式

