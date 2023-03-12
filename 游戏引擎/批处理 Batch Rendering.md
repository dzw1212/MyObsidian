将所有几何体集中到一个顶点缓冲区和索引缓冲区，然后只绘制一次，而不是分开单独地一个个绘制；

# 必要性

在使用粒子系统或复杂ui时，画面中会有数以万计的几何体，倘如分别绘制则会极大地拖累性能降低流畅度；

# 原理


![batchRendering|600](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230125003330.png)

每次Draw的时候，实际上只是把对应的Vertex和Index填入VertexBuffer和IndexBuffer，在场景结束时，再统一调用一次DrawCall；
