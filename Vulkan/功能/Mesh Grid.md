目的是想要绘制一个棋盘样式的网格，作为坐标系用来定位，能够自定义大小、网格数、线条宽度；

# 思路

不同网格数的网格可以去blender之类的软件绘制，不过为了满足可变网格数的需求，可以：
1. 预先创建好不同数量网格的模型，全部载入，根据需求挑选对应的模型；
2. 计算出每个顶点的坐标，再使用某种方法绘制顶点之间的线条；
此处使用第二种方案；

将`Pipeline`中的`LineWidth`设为`DynamicState`，在绘制时实时修改；

# 步骤

## 计算出网格顶点的坐标

```cpp
void VulkanRenderer::CalcMeshGridVertexData()
{
	ASSERT(m_fMeshGridSplit >= 1);
	m_vecMeshGridVertices.clear();
	UINT uiMeshGridSplitPointCount = static_cast<UINT>(m_fMeshGridSplit) + 1;
	m_vecMeshGridVertices.resize(uiMeshGridSplitPointCount * uiMeshGridSplitPointCount);
	float fSplitLen = 1.f / m_fMeshGridSplit;
	for (UINT i = 0; i < uiMeshGridSplitPointCount; ++i)
	{
		for (UINT j = 0; j < uiMeshGridSplitPointCount; ++j)
		{
			UINT uiIdx = i * uiMeshGridSplitPointCount + j;
			float x = static_cast<float>(j) * fSplitLen - 0.5f; 
			float y = static_cast<float>(i) * fSplitLen - 0.5f; //中心点为(0,0)
			m_vecMeshGridVertices[uiIdx].pos = {x, y, 0.f};
		}
	}
}
```

## 绘制顶点之间的线条

选择顶点装配方式为`VK_PRIMITIVE_TOPOLOGY_LINE_LIST`，这种方式会在读取到的每两个顶点之间绘制线条，相比`VK_PRIMITIVE_TOPOLOGY_LINE_STRIP`，占用的`Index Buffer`的空间更大，但也更容易让人理解；

```cpp
VkPipelineInputAssemblyStateCreateInfo inputAssemblyCreateInfo{};
inputAssemblyCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_INPUT_ASSEMBLY_STATE_CREATE_INFO;
inputAssemblyCreateInfo.topology = VK_PRIMITIVE_TOPOLOGY_LINE_LIST;
inputAssemblyCreateInfo.primitiveRestartEnable = VK_FALSE;
```

```cpp
void VulkanRenderer::CalcMeshGridIndexData()
{
	UINT uiVertexCount = m_vecMeshGridVertices.size();
	ASSERT(uiVertexCount >= 4);
	m_vecMeshGridIndices.clear();
	UINT uiMeshGridSplitLineCount = static_cast<UINT>(m_fMeshGridSplit);
	UINT uiMeshGridSplitPointCount = static_cast<UINT>(m_fMeshGridSplit) + 1;
	
	//横向
	for (UINT i = 0; i < uiMeshGridSplitPointCount; ++i)
	{
		for (UINT j = 0; j < uiMeshGridSplitLineCount; ++j)
		{
			UINT uiStart = i * uiMeshGridSplitPointCount + j;
			m_vecMeshGridIndices.push_back(uiStart);
			m_vecMeshGridIndices.push_back(uiStart + 1);
		}
	}

	//纵向
	for (UINT i = 0; i < uiMeshGridSplitPointCount; ++i)
	{
		for (UINT j = 0; j < uiMeshGridSplitLineCount; ++j)
		{
			UINT uiStart = i + j * uiMeshGridSplitPointCount;
			m_vecMeshGridIndices.push_back(uiStart);
			m_vecMeshGridIndices.push_back(uiStart + uiMeshGridSplitPointCount);
		}
	}
}
```


如此，`Vertex`和`Index`数据都已经得到了，只要在网格数量或网格大小修改时更新`VertexBuffer`和`IndexBuffer`即可；

## 将线宽设为动态

```cpp
VkPipelineDynamicStateCreateInfo dynamicStateCreateInfo{};
dynamicStateCreateInfo.sType = VK_STRUCTURE_TYPE_PIPELINE_DYNAMIC_STATE_CREATE_INFO;
std::vector<VkDynamicState> vecDynamicStates = {
	VK_DYNAMIC_STATE_VIEWPORT,
	VK_DYNAMIC_STATE_SCISSOR,
	VK_DYNAMIC_STATE_LINE_WIDTH,
};
dynamicStateCreateInfo.dynamicStateCount = static_cast<UINT>(vecDynamicStates.size());
dynamicStateCreateInfo.pDynamicStates = vecDynamicStates.data();
```

## 渲染网格

```cpp
if (m_bEnableMeshGrid)
{
	if (m_fLastMeshGridSplit != m_fMeshGridSplit)
	{
		RecreateMeshGridVertexBuffer();
		RecreateMeshGridIndexBuffer();
		m_fLastMeshGridSplit = m_fMeshGridSplit;
	}
	if (m_fLastMeshGridSize != m_fMeshGridSize)
	{
		RecreateMeshGridVertexBuffer();
		m_fLastMeshGridSize = m_fMeshGridSize;
	}
	vkCmdSetLineWidth(commandBuffer, m_fMeshGridLineWidth);
	vkCmdBindPipeline(commandBuffer, VK_PIPELINE_BIND_POINT_GRAPHICS, m_MeshGridGraphicPipeline);
	VkBuffer meshGridVertexBuffers[] = {
		m_MeshGridVertexBuffer,
	};
	VkDeviceSize meshGridOffsets[]{ 0 };
	vkCmdBindVertexBuffers(commandBuffer, 0, 1, meshGridVertexBuffers, meshGridOffsets);
	vkCmdBindIndexBuffer(commandBuffer, m_MeshGridIndexBuffer, 0, VK_INDEX_TYPE_UINT32);
	vkCmdBindDescriptorSets(commandBuffer,
		VK_PIPELINE_BIND_POINT_GRAPHICS,
		m_MeshGridGraphicPipelineLayout,
		0, 1,
		&m_vecMeshGridDescriptorSets[uiIdx],
		0, NULL);

	vkCmdDrawIndexed(commandBuffer, static_cast<UINT>(m_vecMeshGridIndices.size()), 1, 0, 0, 0);
	vkCmdSetLineWidth(commandBuffer, 1.f);
}
```


# 效果

![meshGrid|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230723215010.png)

![meshGrid|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230723215046.png)

![meshgrid|800](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20230727021611.png)

# 踩坑

1、`VertexBuffer`和`IndexBuffer`的更新，需要在网格的渲染之前完成，假如通过ImGui的`DragFloat`事件去响应更新，则此时`CommandBuffer`中还残留了已经被销毁的资源，会引起大量的报错，因此使用异步的方式、在检测到网格数和网格尺寸变化时再更新资源；

2、刚开始绘制网格后，发现窗口大小变化时，原本正方形的网格会被拉伸或挤压成矩形，最后发现是对`vkCmdSetViewport`和`vkCmdSetScissor`的理解不到位。类似这种函数，其应用范围是该`CommandBuffer`后续绑定的所有`Pipeline`，因此需要格外注意这类函数在渲染过程中所处的位置；