通过`Unreal Interface`来实现这个功能，只有继承这个接口的Actor才会高亮；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209201318.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209201350.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209223856.png)

`Character`继承这个接口，并实现相关函数：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209223830.png)

# 鼠标射线检测

![900](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251208224419.png)



为了能检测到，还需要将蓝图中的碰撞设置为阻挡`Visibility`：

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209224327.png)

或者在代码中进行设置：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209225009.png)

# 高亮与取消高亮

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251208223335.png)

## 项目启动DepthStencil

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251208234629.png)


## 场景后处理材质

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251208224910.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251208230545.png)


![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251208231433.png)

