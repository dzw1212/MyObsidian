虚幻自5.1起默认启用增强输入系统，代替之前的动作映射与轴映射；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134238.png)

# 定义IA

在`Input/Actions`目录下可以看到当前定义的`InputAction`；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134437.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134454.png)

在详细面板内，可以看到`IA`的类型；
IA_Move为2维数据类型，IA_Jump则为触发条件为Press的bool数据类型；

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134747.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134820.png)

# 配置IMC

在`Input`目录下，有负责映射`IA`的`Input Mapping Context`；`IA`需要在`IMC`中与具体的按键映射才能生效；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210134957.png)

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135009.png)

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135110.png)

通常情况下，按键D代表向右移动，对应x轴的正方向，这也是默认的Input操作，因此按键为D时什么也不做，按键为A时只需要进行一次反转；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210140558.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210140705.png)


# 启用

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135534.png)


# 优先级

事件的传递使分层的，并且具有优先级；
在复杂的游戏场景中可以使用多个`IMC`，并通过优先级控制事件传递顺序；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135729.png)

如果想要`IA`不传递到下一个层级，可以通过`Consume Input`来控制；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20231210135859.png)
