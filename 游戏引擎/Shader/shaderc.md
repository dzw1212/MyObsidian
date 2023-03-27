*https://github.com/google/shaderc*

shaderc是一系列关于shader编译的工具、库和测试样例，包括：
- `glslc`：一个从GLSL/HLSL到SPIR-V的命令行编译器；
- `libshaderc`：一个用于访问glslc的接口库；

shaderc cmd示例：
```cmd
$ shaderc --type vertex -o output.spv input.vert
```

shaderc C++库示例：
```c++
#include <shaderc/shaderc.hpp>
#include <iostream>
#include <fstream>

int main() {
  // 读取 GLSL 代码
  std::ifstream shader_file("shader.vert");
  std::string shader_source((std::istreambuf_iterator<char>(shader_file)), std::istreambuf_iterator<char>());
  
  // 创建 Shaderc 编译器
  shaderc::Compiler compiler;
  
  // 创建编译选项
  shaderc::CompileOptions options;
  options.SetTargetEnvironment(shaderc_target_env_opengl, shaderc_env_version_opengl_4_5);
  options.SetSourceLanguage(shaderc_source_language_glsl);
  options.SetOptimizationLevel(shaderc_optimization_level_zero);
  
  // 编译 GLSL 代码为 SPIR-V
  shaderc::SpvCompilationResult result = compiler.CompileGlslToSpv(shader_source, shaderc_glsl_vertex_shader, "shader.vert", options);
  
  // 检查编译结果
  if (result.GetCompilationStatus() != shaderc_compilation_status_success) {
    std::cerr << result.GetErrorMessage();
    return 1;
  }
  
  // 将 SPIR-V 写入文件
  std::ofstream output_file("output.spv", std::ios::binary);
  output_file.write(result.cbegin(), result.cend());
  
  return 0;
}

```