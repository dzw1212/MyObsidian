OpenGL中有多种屏幕缓冲：
1. 用于写入颜色值的颜色缓冲；
2. 用于写入深度信息的深度缓冲；
3. 允许根据某些条件丢弃特定片段的模板缓冲；
将这些缓冲结合到一起，就成了**帧缓冲 Frame Buffer**；

如果没有显式创建帧缓冲，GLFW会在创建窗口时创建默认帧缓冲；

一个**完整**的帧缓冲需要满足以下条件：
1. 附加至少一个缓冲（颜色、深度或者模板缓冲）；
2. 至少有一个Color Attachment；
3. 所有Attachment都必须是完整的；
4. 每个缓冲都应该有相同的样本数；

# 创建与删除

```c=+
unsigned int fbo;
glGenFramebuffers(1, &fbo);

glDeleteFramebuffers(1, &fbo);
```

# 绑定

```c++
glBindFramebuffer(GL_FRAMEBUFFER, fbo);
```

# 检查帧缓冲是否完整

```c++
if (glCheckFramebufferStatus(GL_FRAMEBUFFER) == GL_FRAMEBUFFER_COMPLETE) {}
```

# 附加附件

[[Attachment]]

## 纹理方式

## 渲染缓冲对象方式

# 渲染到纹理

```c++
glFramebufferRenderbuffer(GL_FRAMEBUFFER, GL_DEPTH_STENCIL_ATTACHMENT, GL_RENDERBUFFER, rbo);
```

