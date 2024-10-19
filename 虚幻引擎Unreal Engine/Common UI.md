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

**Step1、设置视口类，使其支持输入路由**

视口（Viewport） 是Common UI全部输入路由的基础。当Common UI捕获输入时，会将其发送至视口，然后视口会将其发送至顶部的节点；

通用设置中，设置客户端视口类为`CommonGameViewportClient`，如果想用自定义的视口类，需要继承自该通用视口类；

Step2、创建输入动作数据表，将控制器按键映射到UI中

PS：Common UI的输入动作数据表与项目设置中的输入动作或者高级输入系统无关，它们仅用于管理UI输入；


