几何项描述了物体表面几何的自遮挡现象，分为`Shadowing`和`Masking`两种情况；

- Shadowing - 光线无法照射到；

![GDF|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230917141714.png)

- Masking - 反射光线无法射入摄像机；

![GDF|250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230917141828.png)

# Smith Shadowing-Masking

将`Shadowing`与`Masking`分开处理，两者相乘作为最终的子遮挡结果；

$$
G(i,o,m)=G_1(i,m)G_1(o,m)
$$

![G|350](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230917143159.png)

当视线方向与法线重合时，几乎不存在子遮挡的问题；
当视线方向与法线垂直时（掠射角度），几乎完全被遮挡；