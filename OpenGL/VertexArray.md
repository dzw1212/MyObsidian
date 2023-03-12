
```c++
glGenVertexArrays(1, &m_uiId);

glBindVertexArray(m_uiId);

for (BYTE idx = 0; idx < layout.GetElements().size(); ++idx)
{
	auto& element = layout.GetElements()[idx];
	glEnableVertexAttribArray(idx);
	glVertexAttribPointer(idx,
		element.GetComponentCount(),
		ShaderDataTypeToOpenGLBaseType(element.eType),
		element.bNormalized ? GL_TRUE : GL_FALSE,
		layout.GetStride(),
		(const void*)element.uiOffset);
}

glDeleteVertexArrays(1, &m_uiId);
```

# 创建与删除

```c++
glGenVertexArrays(1, &m_uiId);

glDeleteVertexArrays(1, &m_uiId);
```

# 绑定

```c++
glBindVertexArray(m_uiId);
```

# 启用顶点属性

```c++
glEnableVertexAttribArray(idx); //与VertexShader中layout(location=n)对应
```

# 链接顶点属性

```c++
glVertexAttribPointer(
	0, //顶点属性的位置，与VetexShader中layout(location=n)对应
	3, //顶点属性的元素个数，比如vec3就是3
	GL_FLOAT, //顶点属性的数据类型
	GL_FALSE, //是否标准化，若为true，所有数据会被映射到(0,1)之间
	3 * sizeof(float), //连续的两个顶点属性之间的步长Stride
	(void*)0); //顶点属性的偏移量
```
