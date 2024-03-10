附件Attachment表示一个内存位置，能够作为FrameBuffer的一个缓冲，可以将其想象成一个图像；

[[FrameBuffer]]

# Color Attachment

## 纹理 Texture

当把一个Texture附加给一个FrameBuffer时，所有的渲染指令都会写入这个纹理中，因此所有渲染操作的结果都会被存储在该纹理中，方便之后的使用；

### 创建

创建ColorAttachment与创建普通的Texture类似；
```c++
unsigned int texture;
glGenTextures(1, &texture);
glBindTexture(GL_TEXTURE_2D, texture);

glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, 800, 600, 0, GL_RGB, GL_UNSIGNED_BYTE, nullptr);

glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

对于这个新创建的Texture，一般大小设为屏幕的大小，只分配内存但不进行填充；

### 附加

```c++
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_COLOR_ATTACHMENT0, GL_TEXTURE_2D, texture, 0);
```


## 深度Depth

### 创建

### 附加

```c++
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_ATTACHMENT, GL_DEPTH_COMPONENT, texture, 0);
```

## 模板 Stencil

### 创建

### 附加

```c++
glFramebufferTexture2D(GL_FRAMEBUFFER, GL_STENCIL_ATTACHMENT, GL_STENCIL_IDX, texture, 0);
```

## 合并Depth与Stencil

```c++
//纹理的每32位数据包含24位深度信息与8位模板信息

glTexImage2D(
  GL_TEXTURE_2D, 0, GL_DEPTH24_STENCIL8, 800, 600, 0, 
  GL_DEPTH_STENCIL, GL_UNSIGNED_INT_24_8, NULL
);

glFramebufferTexture2D(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_TEXTURE_2D, texture, 0);
```

# RenderBufferObject Attachment

在过去，Color Attachment是唯一可用的附件，之后新加入了专门为FrameBuffer设计的渲染缓冲对象附件；

## 什么时候用

通常的规则是，如果你不需要从一个缓冲中采样数据，那么对这个缓冲使用渲染缓冲对象会是明智的选择。如果你需要从缓冲中采样颜色或深度值等数据，那么你应该选择纹理附件；

## 优点

渲染缓冲对象会将数据储存为OpenGL原生的渲染格式，不会做任何针对纹理格式的转换，使其变为一个更快的可写储存介质；

## 创建

```c++
unsigned int rbo;
glGenRenderbuffers(1, &rbo);
```

## 绑定

```c++
glRenderbufferStorage(GL_RENDERBUFFER, GL_DEPTH24_STENCIL8, 800, 600);
```

