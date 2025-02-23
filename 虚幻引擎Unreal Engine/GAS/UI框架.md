使用经典的MVC框架来分离逻辑；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250213234750.png)


![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250213235003.png)

---

`WidgetController`负责获取数据，并提供接口给下属的`UserWidget`使用；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250215163402.png)



`Game Mode`的`Hud`一般用来控制UI的创建、布局等；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250215141238.png)

新建一个`Hud`，用来管理各个`Widget`与`WidgetController`；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250215163521.png)

推荐在`ASC`系统`Init Actor Info`之后立即初始化`WidgetController`；

![850](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250215163629.png)

# Attribute改变通知UI

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250215193121.png)

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250215193140.png)

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250215193427.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250215193447.png)

控件蓝图中`Assign Delegate`事件：

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250215193603.png)

# Attribute响应架构

直观的做法是，为每个`Attribite`都设置其对应的`Delegate`与响应事件，但是这种做法随着`Attribute`数量增多，会给代码的可维护性与简洁性带来挑战；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250221011511.png)

或者将所有的属性修改打包为一个结构体，然后通过`Tag`区分是哪个`Attribute`发生了变化、变化了多少，并且通过`DT`或`DA`建立控件与该结构体之间的对应关系；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250221011850.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250221012125.png)

步骤：

1. 为了避免在cpp中频繁通过`RequestGameplayTag`来获取，直接在cpp中创建`Native GameplayTag`；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250221012303.png)

创建一个单例`Native GameplayTag`用来创建`Attribute`对应的`Tag`，并且在自定义的`AssetManager`中初始化；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250222173711.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250222173930.png)

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250222180813.png)


然后在`/Config/DefaultEngine.ini`中设置`AssetManager`类：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250222174058.png)

打开编辑器，可以看到`Native GameplayTag`成功添加：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250222174132.png)

至此之后，在cpp中任意处可以通过`FAuraGameplayTags::Get().Attributes_Secondary_XXX`来获取标签；

2. 每个UI控件有自己单独的Tag，当接受到事件后，会比对Tag，如果对应上了再进行下一步操作；

![800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250223155708.png)

