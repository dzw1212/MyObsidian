
# 创建Program

```c++
GLuint program = glCreateProgram();
```

# 创建Shader

```c++
// Create an empty shader handle
GLuint shaderId = glCreateShader(shaderType);
```

# 删除Shader

```c++
glDeleteShader(shaderId);
```

# 载入Shader Code

```c++
// Send the shader source code to GL
// Note that std::string's .c_str is NULL character terminated.
const GLchar* source = (const GLchar*)strShader.c_str();
glShaderSource(shaderId, 1, &source, 0);
```

# 编译Shader

```c++
// Compile the shader
glCompileShader(shaderId);
```

## 查看编译结果

```c++
GLint isCompiled = 0;
glGetShaderiv(shaderId, GL_COMPILE_STATUS, &isCompiled);
if (isCompiled == GL_FALSE)
{
	GLint maxLength = 0;
	glGetShaderiv(shaderId, GL_INFO_LOG_LENGTH, &maxLength);

	// The maxLength includes the NULL character
	std::vector<GLchar> infoLog(maxLength);
	glGetShaderInfoLog(shaderId, maxLength, &maxLength, &infoLog[0]);

	// We don't need the shader anymore.
	glDeleteShader(shaderId);

	// Use the infoLog as you see fit.
	CORE_ASSERT(false, std::format("Compile {} Shader Failed, Info is {}", ShaderTypeToString(shaderType), infoLog.data()));

	break;
}
```

# 附加Shader到Program

```c++
// Attach our shaders to our program
glAttachShader(program, shaderId);
```

# 分离Shader

```c++
glDetachShader(program, shaderId);
```

# 链接Program

```c++
// Link our program
glLinkProgram(program);
```

## 查看链接结果

```c++
GLint isLinked = 0;
glGetProgramiv(program, GL_LINK_STATUS, (int*)&isLinked);
if (isLinked == GL_FALSE)
{
	GLint maxLength = 0;
	glGetProgramiv(program, GL_INFO_LOG_LENGTH, &maxLength);

	// The maxLength includes the NULL character
	std::vector<GLchar> infoLog(maxLength);
	glGetProgramInfoLog(program, maxLength, &maxLength, &infoLog[0]);

	// We don't need the program anymore.
	glDeleteProgram(program);
	// Don't leak shaders either.
	for (auto shaderId : vecShaderId)
		glDeleteShader(shaderId);

	// Use the infoLog as you see fit.
	CORE_ASSERT(false, std::format("Link Shader Failed, Info is {}", infoLog.data()));
}
```

# Uniform变量

```c++
//uniform mat4 u_ViewProjMat;

auto location = glGetUniformLocation(m_uiRendererProgramId, "u_ViewProjMat");
glUniform4f(location, fvec4.x, fvec4.y, fvec4.z, fvec4.w);
```

# 绑定Program

```c++
glUseProgram(m_uiRendererProgramId);
```

# 示例代码

```c++

//mapShaderSrc中，
//键为ShaderType（GL_VERTEX_SHADER等），值为ShaderCode

GLuint program = glCreateProgram();
std::vector<GLuint> vecShaderId;

for (auto& iter : mapShaderSrc)
{
	GLenum shaderType = iter.first;
	const std::string& strShader = iter.second;

	// Create an empty shader handle
	GLuint shaderId = glCreateShader(shaderType);

	// Send the shader source code to GL
	// Note that std::string's .c_str is NULL character terminated.
	const GLchar* source = (const GLchar*)strShader.c_str();
	glShaderSource(shaderId, 1, &source, 0);

	// Compile the shader
	glCompileShader(shaderId);

	GLint isCompiled = 0;
	glGetShaderiv(shaderId, GL_COMPILE_STATUS, &isCompiled);
	if (isCompiled == GL_FALSE)
	{
		GLint maxLength = 0;
		glGetShaderiv(shaderId, GL_INFO_LOG_LENGTH, &maxLength);

		// The maxLength includes the NULL character
		std::vector<GLchar> infoLog(maxLength);
		glGetShaderInfoLog(shaderId, maxLength, &maxLength, &infoLog[0]);

		// We don't need the shader anymore.
		glDeleteShader(shaderId);

		// Use the infoLog as you see fit.
		CORE_ASSERT(false, std::format("Compile {} Shader Failed, Info is {}", ShaderTypeToString(shaderType), infoLog.data()));

		break;
	}
	
		// Attach our shaders to our program
		glAttachShader(program, shaderId);
	
		vecShaderId.push_back(shaderId);
	}
	
	// Link our program
	glLinkProgram(program);
	
	// Note the different functions here: glGetProgram* instead of glGetShader*.
	GLint isLinked = 0;
	glGetProgramiv(program, GL_LINK_STATUS, (int*)&isLinked);
	if (isLinked == GL_FALSE)
	{
		GLint maxLength = 0;
		glGetProgramiv(program, GL_INFO_LOG_LENGTH, &maxLength);
	
		// The maxLength includes the NULL character
		std::vector<GLchar> infoLog(maxLength);
		glGetProgramInfoLog(program, maxLength, &maxLength, &infoLog[0]);
	
		// We don't need the program anymore.
		glDeleteProgram(program);
		// Don't leak shaders either.
		for (auto shaderId : vecShaderId)
			glDeleteShader(shaderId);
	
		// Use the infoLog as you see fit.
		CORE_ASSERT(false, std::format("Link Shader Failed, Info is {}", infoLog.data()));
	}
	
	// Always detach shaders after a successful link.
	for (auto shaderId : vecShaderId)
		glDetachShader(program, shaderId);
	
	m_uiRendererProgramId = program;
}
```

# GLSL语法

## 分量重组

```c
vec2 someVec;
vec4 differentVec = someVec.xyxx;
vec3 anotherVec = differentVec.zyw;
vec4 otherVec = someVec.xxxx + anotherVec.yxzy;
```

```c
vec2 vect = vec2(0.5, 0.7);
vec4 result = vec4(vect, 0.0, 0.0);
vec4 otherResult = vec4(result.xyz, 1.0);
```

## 输入与输出

各个Shader组成一个Pipeline，Shader之间存在数据的传递；

```c
//Vertex Shader接收程序的数据
layout(location = 0) in vec3 vertPos;
layout(location = 1) in vec2 texCoord;
layout(location = 2) in vec4 color;
layout(location = 3) in float texIndex;
layout(location = 4) in float tilingFactor;

//Fragment Shader向程序传递数据
layout(location = 0) out vec4 color;
```

```c
//Vertex Shader传递数据给Fragment Shader，变量名与类型需要对应
out vec2 v_TexCoord;
out vec4 v_Color;
out float v_TexIndex;
out float v_TilingFactor;

in vec2 v_TexCoord;
in vec4 v_Color;
in float v_TexIndex;
in float v_TilingFactor;
```

