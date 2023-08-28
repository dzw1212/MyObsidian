
不同图形API的坐标系不同，这里指的是与硬件相关的NDC坐标系与FrameBuffer坐标系，至于world坐标系和view坐标系，则是由用户自行定义；

# NDC坐标系

![ndc|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230816171419.png)

# FrameBuffer坐标系

![frameBuffer|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230816171733.png)


# FlipY方案

1. Shader中直接修改（Dirty）；

```glsl
gl_Position.y *= -1.0;
```

2. 投影矩阵中修改；

```cpp
proj[1][1] *= -1.f;
```

3. 使用Vulkan拓展，允许viewport中出现负值；



# 参考

*https://www.saschawillems.de/blog/2019/03/29/flipping-the-vulkan-viewport/*
*https://zhuanlan.zhihu.com/p/574067652*
*https://zhuanlan.zhihu.com/p/339295068*

