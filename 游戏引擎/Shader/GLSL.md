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

## 子程序

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

在应用端选择使用哪个option：

# GLSL for Vulkan

*https://web.engr.oregonstate.edu/~mjb/cs557/Handouts/VulkanGLSL.1pp.pdf

