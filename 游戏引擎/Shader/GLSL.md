# 基础

## 数据类型

基本类型：`float`，`double`，`bool`，`int`，`uint`；

向量：`vec{2,3,4}`，其他类似的有`dvec`，`bvec`，`ivec`，`uvec`

矩阵：
	方阵：`mat2/dmat2`，`mat3/dmat3`，`mat4/dmat4`；
	非方阵：`mat{2,3,4}x{2,3,4}`，`dmat{2,3,4}x{2,3,4}`；

采样器：
	`sampler1D`，`sampler2D`，`sampler3D`；
	`samplerCube`，用于Cube Map贴图；
	`sampler2DShadow`，用于Shadow map；

原子计数器：支持原子操作；
	*http://www.lighthouse3d.com/tutorials/opengl-atomic-counters/

### 类型强转

```c
int a = 2;
float c = float(a); //2.0
```

### 类型组合

```c
vec2 a = vec2(1.0, 2.0);
vec2 b = vec2(3.0, 4.0);
vec4 c = vec4(a, b); //(1.0, 2.0, 3.0, 4.0)

mat2 n = mat2(a, b); //以列主序的方式填充
```

### 类型拆分

```c
vec4 a = vec4(1.0, 2.0, 3.0, 4.0);

float posX = a.x;
vec2 posXY = a.xy;
float depth = a.w;

//类似于xyzw，颜色的rgba和贴图的stpq也适用（原本是strq，但r被颜色占用了）
```

## 语法

`if/else`，`for(...)`，`while(...)`，`do ... while(...)`；

关键词：
	`continue`
	`break`
	`discard`：只能用于fragment shader，直接terminate，不写入framebuffer；

修饰词：
	`in`：输入参数（若无修饰词则默认）；
	`out`：输出参数；
	`inout`：既可以输入也可以输出；


btw：支持参数不同的同名函数重载，不支持递归；

## 子程序 Subroutine

对于需要根据某个变量的值来进行分支处理的情况，可以借助子程序`subroutine`；
提升了代码整洁程度和执行效率，但是可读性和debug能力也降低了；

subroutine根据一个变量的值，将一个函数调用绑定到一系列函数定义上，此时一个uniform变量被当成一个函数指针，并且可以被用于调用一个函数；

### 示例

原函数：
```c
layout (std140) uniform Matrixs {
	mat4 pvm;
};

in vec4 position;
out vec4 color;

uniform int redBlueFlag;

void main()
{
	if (redBlueFlag == 1)
		color = vec4(1.0, 0.0, 0.0, 1.0);
	else
		color = vec4(0.0, 0.0, 1.0, 1.0);
	gl_Position = pvm * position;
}
```

使用subroutine后：
```c
layout (std140) uniform Matrixs {
	mat4 pvm;
};

in vec4 position;
out vec4 color;

uniform int redBlueFlag;

subroutine vec4 colorRedBlue(); //定义subroutine signature

subroutine (colorRedBule) vec4 RedColor() { //option 1
	return vec4(1.0, 0.0, 0.0, 1.0);
}

subroutine (colorRedBule) vec4 BlueColor() { //option 2
	return vec4(0.0, 0.0, 1.0, 1.0);
}

subroutine uniform colorRedBlue myRedBlueSelection;

void main()
{
	color = myRedBlueSelection();
	gl_Position = pvm * position;
}
```

在应用端选择使用哪个option；


## 内建变量 Build-in Variable

### 顶点着色器变量

- `gl_Position`：顶点着色器的输出变量，设置顶点的位置；

- `gl_PointSize`：顶点着色器的输出变量，设置顶点作为图元时的大小；
	如果使用图元`GL_POINTS`的话，每一个点都是一个图元，会被渲染为一个点；
	点的大小有两种设置方法：
	1. 可以通过`glPointSize`函数来设置渲染出的点的大小；
	2. 在顶点着色器中修改gl_PointSize变量的值（该功能默认关闭）；
```c
glEnable(GL_PROGRAM_POINT_SIZE); //开启gl_PointSize修改点大小的功能；
```

- `gl_VertexID`：顶点着色器的输入变量，存储当前正在绘制的顶点的ID；
	当使用`glDrawElements`进行索引绘制的时候，存储当前顶点的索引；
	当使用`glDrawArrays`进行绘制的时候，存储从DrawCall开始的已处理的顶点数量；

### 片段着色器变量

- `gl_FragCoord`：片段着色器的输入变量，表示当前片元的坐标；
	原点位于窗口的左下角，z值可以用来做深度判断；

- `gl_FragFacing`：片段着色器的输入变量，表示一个片元的朝向；
	若为true，则当前片元是正向面的一部分；若为false，则当前片元是反向面的一部分；
	朝向的正反由顶点的环绕顺序决定；
	如果开启了面剔除，则看不到反向的面，朝向也就没有意义了；

- `gl_FragDepth`：片段着色器的输出变量，能修改当前片元的深度值；
	gl_FragDepth可以设为0.0到1.0之间的任意值，默认为gl_FragCooed.z；
	但是直接使用gl_FragDepth会带来性能问题 --- OpenGL会禁用所有的==Early Depth Testing==；
	或者说使用==Depth Condition==来声明gl_FragDepth变量（v4.2起支持）；
```c
layout(depth_<condition>) out float gl_FragDepth;
```
其中condition：
1> **any**：默认值，禁用Early Depth Testing，影响性能；
2> **greater**：深度值只能比gl_FragCoord.z更大；
3> **less**：深度值只能比gl_FragCoord.z更小；
4> **unchaned**：即使要设置gl_FragDepth，也只能设为gl_FragCoord.z；


## 接口块 Interface Block

使用接口块来管理着色器之间数据的传输，替代之前零散定于in/out的方式；
接口块的声明与结构体类似，使用`in`和`out`关键字来定义；
相当于就是一个限制了名称的结构体，要求着色器之间块名一致；
```c
//Vertex Shader
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoords;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

out VS_OUT //块名为VS_OUT，实例名为vs_out
{
    vec2 TexCoords;
} vs_out;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);    
    vs_out.TexCoords = aTexCoords;
}
```

```c
//Fragment Shader
#version 330 core
out vec4 FragColor;

in VS_OUT //块名需要对应一致，实例名随意
{
    vec2 TexCoords;
} fs_in;

uniform sampler2D texture;

void main()
{             
    FragColor = texture(texture, fs_in.TexCoords);   
}
```

## Uniform缓冲对象 UBO

当存在多个Shader的时候，会发现一种情况：每个Shader中的uniform变量都是相同的（或者说重合的），但对于每个Shader还要单独的设置uniform变量，这是一种浪费；

此时可以使用Uniform Buffer Object，将所有uniform变量存储于该缓冲中，需要时直接从缓冲中提取即可；

与其他缓冲类似，使用`glGenBuffers`来创建UBO，并绑定为`GL_UNIFORM_BUFFER`，将uniform变量写入该缓冲；
```c
#version 330 core
layout (location = 0) in vec3 aPos;

layout (std140) uniform Matrices //UBO，设置内存布局为std140
{
    mat4 projection;
    mat4 view;
};

uniform mat4 model;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
    //UBO中的变量可以直接访问，不需要加前缀
}
```

### Uniform块布局 UBL

uniform块只是一块预留内存，本身不会保存数据类型信息，需要将uniform变量与内存对应；
```c
layout (std140) uniform ExampleBlock
{
    float value;
    vec3  vector;
    mat4  matrix;
    float values[3];
    bool  boolean;
    int   integer;
};
```

对于简单类型（float/bool/int等）其元素大小是明确的，但对于复杂类型（vec/mat）其float元素的间距是不明确的，允许硬件能够在它认为合适的位置放置；

#### shared布局

默认情况下，GLSL会使用**共享布局 Shared Layout**，一旦硬件定义了偏移量，则偏移在多个程序中是共享且一致的；==使用共享布局时，GLSL可以为了优化而调整uniform变量的位置==，因此除非使用`glGetUniformIndices`来查询，否则无法得知每个uniform变量的位置；

#### std140布局

或者使用`std140`布局，std140布局声明了每个变量的偏移量都是由一系列规则所决定的，根据这个规则，我们可以==手动计算每个变量的偏移量==；
**基准对齐量 Base Alignment**：等于一个变量在uniform块中所占据的空间；
**对齐偏移量 Aligned Offset**：表示一个变量从块起始位置起的字节偏移量，一个变量的对齐偏移量必须是基准对齐量的倍数；

布局规则：*https://registry.khronos.org/OpenGL/extensions/ARB/ARB_uniform_buffer_object.txt*
布局实例：
```c
layout (std140) uniform ExampleBlock
{
                     // 基准对齐量       // 对齐偏移量
    float value;     // 4               // 0 
    vec3 vector;     // 16              // 16  (必须是16的倍数，所以 4->16)
    mat4 matrix;     // 16              // 32  (列 0)
                     // 16              // 48  (列 1)
                     // 16              // 64  (列 2)
                     // 16              // 80  (列 3)
    float values[3]; // 16              // 96  (values[0])
                     // 16              // 112 (values[1])
                     // 16              // 128 (values[2])
    bool boolean;    // 4               // 144
    int integer;     // 4               // 148
}; 
```

在得知对齐偏移量之后，就可以使用`glBufferSubData`将uniform变量写入UBO了；

#### packed布局

==紧凑布局的效率是最高的==，但是当使用紧凑布局时，无法保证该布局在不同程序不同硬件下是不变的，并且其允许编译器将uniform变量从uniform块中优化掉；

### UBO的使用方式

#### 创建

```c
unsigned int uboExampleBlock;
glGenBuffers(1, &uboExampleBlock);
```

#### 绑定

```c
glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
```

#### 分配内存大小

```c
glBufferData(GL_UNIFORM_BUFFER, 152, NULL, GL_STATIC_DRAW); // 分配152字节的内存
```

#### 插入数据

Step1. 将uniform块绑定到一个特定的绑定点上

方法一、使用`glUniformBlockBinding`+`glGetUniformBlockIndex`；
```c
unsigned int lights_index = glGetUniformBlockIndex(shaderA.ID, "Lights");   
glUniformBlockBinding(shaderA.ID, lights_index, 2); 
//将Lights对应的uniform块链接到绑定点2
//需要对每个着色器重复该步骤
```

方法二、使用布局标识符`binding`；
```c
layout(std140, binding = 2) uniform Lights { ... };
```

Step2. 绑定uniform缓冲对象到绑定点上；

```c
glBindBufferBase(GL_UNIFORM_BUFFER, 2, uboExampleBlock); 
// 或
glBindBufferRange(GL_UNIFORM_BUFFER, 2, uboExampleBlock, 0, 152); 
//支持只绑定一部分，让多个uniform块绑定到同一个uniform缓冲对象上
```

Step3. 向uniform缓冲中添加数据；

```c
glBindBuffer(GL_UNIFORM_BUFFER, uboExampleBlock);
int b = true; // GLSL中的bool是4字节的，所以我们将它存为一个integer
glBufferSubData(GL_UNIFORM_BUFFER, 144, 4, &b); 
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

### UBO示例

```c
#version 330 core
layout (location = 0) in vec3 aPos;

layout (std140) uniform Matrices //proj和view不常变化，因此使用UBO
{
    mat4 projection;
    mat4 view;
};
uniform mat4 model;

void main()
{
    gl_Position = projection * view * model * vec4(aPos, 1.0);
}
```

```c
//将四个Shader中的Matrices块都链接到绑定点0上
unsigned int uniformBlockIndexRed    = glGetUniformBlockIndex(shaderRed.ID, "Matrices");
unsigned int uniformBlockIndexGreen  = glGetUniformBlockIndex(shaderGreen.ID, "Matrices");
unsigned int uniformBlockIndexBlue   = glGetUniformBlockIndex(shaderBlue.ID, "Matrices");
unsigned int uniformBlockIndexYellow = glGetUniformBlockIndex(shaderYellow.ID, "Matrices");  

glUniformBlockBinding(shaderRed.ID,    uniformBlockIndexRed, 0);
glUniformBlockBinding(shaderGreen.ID,  uniformBlockIndexGreen, 0);
glUniformBlockBinding(shaderBlue.ID,   uniformBlockIndexBlue, 0);
glUniformBlockBinding(shaderYellow.ID, uniformBlockIndexYellow, 0);
```

```c
//创建UBO并绑定到绑定点0上，指定范围
unsigned int uboMatrices
glGenBuffers(1, &uboMatrices);

glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
glBufferData(GL_UNIFORM_BUFFER, 2 * sizeof(glm::mat4), NULL, GL_STATIC_DRAW);
glBindBuffer(GL_UNIFORM_BUFFER, 0);

glBindBufferRange(GL_UNIFORM_BUFFER, 0, uboMatrices, 0, 2 * sizeof(glm::mat4));
```

```c
//填充proj matrix
glm::mat4 projection = glm::perspective(glm::radians(45.0f), (float)width/(float)height, 0.1f, 100.0f);
glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
glBufferSubData(GL_UNIFORM_BUFFER, 0, sizeof(glm::mat4), glm::value_ptr(projection));
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

```c
//填充view matrix
glm::mat4 view = camera.GetViewMatrix();           
glBindBuffer(GL_UNIFORM_BUFFER, uboMatrices);
glBufferSubData(GL_UNIFORM_BUFFER, sizeof(glm::mat4), sizeof(glm::mat4), glm::value_ptr(view));
glBindBuffer(GL_UNIFORM_BUFFER, 0);
```

```c
//绘制
glBindVertexArray(cubeVAO);
shaderRed.use();
glm::mat4 model;
model = glm::translate(model, glm::vec3(-0.75f, 0.75f, 0.0f));  // 移动到左上角
shaderRed.setMat4("model", model);
glDrawArrays(GL_TRIANGLES, 0, 36);        
// ... 绘制绿色立方体
// ... 绘制蓝色立方体
// ... 绘制黄色立方体 
```

### 优点

1. 一次设置很多uniform比一个一个设置要快很多；
2. 相比在多个Shader中修改同样的uniform变量，在UBO中修改一次更高效；
3. 由于OpenGL限制了uniform的数量，因此使用UBO能够使用更多的uniform；


# GLSL for Vulkan

*https://web.engr.oregonstate.edu/~mjb/cs557/Handouts/VulkanGLSL.1pp.pdf

