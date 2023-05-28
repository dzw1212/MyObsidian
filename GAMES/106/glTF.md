https://github.com/KhronosGroup/glTF

https://sketchfab.com/3d-models/buster-drone-294e79652f494130ad2ab00a13fdbafd

`glTF`是一种开放的3D文件格式，它的全称是`GL Transmission Format`。它的设计目的是为了在不同的3D应用程序之间交换3D模型和场景数据。它是由Khronos Group开发的，这是一个非营利性技术联盟，致力于制定开放标准，以促进图形和多媒体技术的发展。gltf文件是基于JSON格式的，可以包含3D模型的几何形状、材质、动画、相机和灯光等信息。它还支持嵌入式纹理和二进制数据，以提高性能。总之，gltf是一种非常有用的3D文件格式，可以帮助开发人员更轻松地在不同的3D应用程序之间共享和交换3D模型和场景数据。

# 顶点蒙皮动画

顶点蒙皮是一种技术，根据骨骼组成的骨架的当前姿势，使用每个顶点的权重；
在渲染过程中，这些骨骼的矩阵和顶点权重被用来计算最终的顶点位置，然后将动画应用于这些骨骼；

## 数据结构

### 节点

```cpp
struct Node
{
  Node *              parent;
  uint32_t            index;
  std::vector<Node *> children;
  Mesh                mesh;
  glm::vec3           translation{};
  glm::vec3           scale{1.0f};
  glm::quat           rotation{};
  int32_t             skin = -1; //应用于该节点的皮肤的索引
  glm::mat4           matrix;
  glm::mat4           getLocalMatrix(); //从初始矩阵和当前分量中计算出本地矩阵
};
```

### 顶点

```cpp
struct Vertex
{
...
	glm::vec4 jointIndices;
	glm::vec4 jointWeights;
};
```

为了计算应用于顶点的最终矩阵，传递关节的下标与权重，这决定了这个顶点收关节影响的程度（glTF支持每个关节最多四个下标和权重）；

### 皮肤

```cpp
struct Skin
{
  std::string            name;
  Node *                 skeletonRoot = nullptr;
  std::vector<glm::mat4> inverseBindMatrices; //用于将几何体转换到相应节点空间
  std::vector<Node *>    joints;
  vks::Buffer            ssbo;
  VkDescriptorSet        descriptorSet;
};
```

### 动画

#### 取样器

```cpp
struct AnimationSampler
{
  std::string            interpolation;
  std::vector<float>     inputs;
  std::vector<glm::vec4> outputsVec4;
};
```

动画采样器包含使用访问器从缓冲区读取的关键帧数据以及关键帧的插值方式。这可以是LINEAR - 一个简单的线性插值，STEP - 在到达下一个关键帧之前保持不变，以及CUBICSPLINE - 使用一个带有切线的立方样条来计算插值的关键帧。

#### 通道

```cpp
struct AnimationChannel
{
  std::string path;
  Node *      node;
  uint32_t    samplerIndex;
};
```

动画通道将节点与动画采样器指定的关键帧连接起来，路径成员指定要动画化的节点属性，可以是平移、旋转、缩放或重量。后者指的是变形目标，而不是顶点权重（用于换肤），在这个例子中没有使用。

#### 动画

```cpp
struct Animation
{
  std::string                   name;
  std::vector<AnimationSampler> samplers;
  std::vector<AnimationChannel> channels;
  float                         start       = std::numeric_limits<float>::max();
  float                         end         = std::numeric_limits<float>::min();
  float                         currentTime = 0.0f;
};
```

## 加载与传递数据

需要使用`VulkanglTFModel::loadNode`的glTF访问器加载关节下标`Joint Index`与关节权重`Joint Weight`；

```cpp
// Get vertex joint indices
if (glTFPrimitive.attributes.find("JOINTS_0") != glTFPrimitive.attributes.end())
{
  const tinygltf::Accessor &  accessor = input.accessors[glTFPrimitive.attributes.find("JOINTS_0")->second];
  const tinygltf::BufferView &view     = input.bufferViews[accessor.bufferView];
  jointIndicesBuffer                   = reinterpret_cast<const uint16_t *>(&(input.buffers[view.buffer].data[accessor.byteOffset + view.byteOffset]));
}
// Get vertex joint weights
if (glTFPrimitive.attributes.find("WEIGHTS_0") != glTFPrimitive.attributes.end())
{
  const tinygltf::Accessor &  accessor = input.accessors[glTFPrimitive.attributes.find("WEIGHTS_0")->second];
  const tinygltf::BufferView &view     = input.bufferViews[accessor.bufferView];
  jointWeightsBuffer                   = reinterpret_cast<const float *>(&(input.buffers[view.buffer].data[accessor.byteOffset + view.byteOffset]));
}
```