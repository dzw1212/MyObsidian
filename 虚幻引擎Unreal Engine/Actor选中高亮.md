通过`Unreal Interface`来实现这个功能，只有继承这个接口的Actor才会高亮；

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209201318.png)

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209201350.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209223856.png)

`Character`继承这个接口，并实现相关函数：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209223830.png)

检测鼠标当前选中的`Actor`：

![550](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209224105.png)

为了能检测到，还需要将蓝图中的碰撞设置为阻挡`Visibility`：

![450](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209224327.png)

或者在代码中进行设置：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20250209225009.png)

