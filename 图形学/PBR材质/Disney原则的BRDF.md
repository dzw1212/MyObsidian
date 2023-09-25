# 为什么不用微表面模型？

1. 微表面模型无法很好地拟合现实中的一些材质（比如缺少Diffuse项）；
2. 微表面模型的参数都是一些不直观的物理量，不方便使用者调参（`Not Artist Friendly`）；

# 设计原则

1. 参数直观，方便调节；
2. 参数尽可能的少；
3. 参数最好限制在0~1的范围内；
4. 参数允许超出范围以实现某些特殊效果；
5. 各种参数能够叠加作用；
6. 不一定严格基于物理；
7. ...

# 参数

![DisneyBRDF|700](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230924203851.png)


- subsurface：次表面散射；
- metallic：金属度；
- specular：镜面反射；
- speculatTint：镜面反射颜色；
- roughness：粗糙度；
- anisotropic：各向异性程度；
- sheen：表面绒毛；
- sheenTine：表面绒毛颜色；
- clearcoat：清漆度；
- clearcoatGloss：清漆光滑程度；

