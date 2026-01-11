AnimLayer就相当于动画节点的函数封装，ALI则像其他Interface一样，是许多AnimLayer的集合；

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20251228205141.png)

比如说，我的主动画蓝图是`ABP_ALS`，它不负责实现ALI中定义的接口，只负责调用；

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109015944.png)

新建一个`Linked Anim Class`，它负责实现ALI的接口：

![700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109020106.png)

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109020132.png)

---

在`ABP_Layer`中访问`ABP_ALS`中的变量的推荐做法：
	*需要注意线程安全问题*；

`ABP_Layer`中存储一个`ABP_ALS`的实例变量，在`Blueprint Thread Safe Update` 中更新；

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109020346.png)

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109020447.png)

然后通过一个`Pure`+`Thread Safe`的接口来获取这个`ABP_ALS`的引用；

![300](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109020606.png)

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109020624.png)

其他地方再通过`Property Access`来获取`ABP_ALS`中的变量：

![500](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109020719.png)

---

`Blend Space`内部的机制可能特殊一些，直接使用`Property Access`获取值的话，会在关闭PIE时产生获取到空值的报错，为了避免这个报错，采用以下写法：

![600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109023207.png)

在线程安全Update中：

![1000](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20260109023327.png)
