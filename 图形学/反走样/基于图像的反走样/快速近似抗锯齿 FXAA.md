`FXAA`（`Fast Approximate Anti-Aliasing`）是一种在计算机图形学中常用的反走样技术，由`NVIDIA`开发的，主要用于抗锯齿；

优点：计算速度快，对硬件要求较低；
缺点：使图像的细节变得模糊；

原理：在图像渲染完成后进行后处理，通过**检测像素的颜色变化**来确定哪些地方可能存在锯齿，然后对这些地方进行平滑处理；

