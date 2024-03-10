*使用VertexBuffer可以一次性发送一大批数据到显卡*；

# 创建

```c++
glCreateBuffers(1, &m_uiId);
```

# 绑定 

```c++
glBindBuffer(GL_ARRAY_BUFFER, m_uiId);
```

# 传递数据

```c++
glBufferData(GL_ARRAY_BUFFER, uiSize, pfVertices, GL_STATIC_DRAW);
```

## 数据管理方式

`GL_STATIC_DRAW` : 数据不会或者几乎不会改变；
`GL_DYNAMIC_DRAW` : 数据会被改变很多；
`GL_STREAM_DRAW` : 每次DrawCall时数据都会改变；

如果是后两种方式，显卡会把VertexBuffer数据存放在支持高速读写的内存部分；