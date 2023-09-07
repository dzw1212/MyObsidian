`Reflective Shadow Map`用于实现全局光照效果，特别是间接光照和柔和阴影；
这种技术通过将光源视角的深度图（shadow map）与反射向量（reflective vector）结合，从而模拟光线在场景中的反射；

# 思想

将直接光照能照射到的每个`shading Point`都视作`Secondary Light Source`；

![RSM|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230905224202.png)

![RSM|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230905224411.png)


为此需要知道两点：
1、哪些点是能被光源直接照射到、可以视作二次光源？
2、这些二次光源对P点所接收光照的权重是多少？

问题一：
	使用`Shadow Map`，每个深度图上的像素都可以视作二次光源；
问题二：
	为了简化计算，首先假设每个二次光源点都是diffuse的（从而保证无论点P在哪，收到的反射光都是一样的）；



