*https://github.com/KhronosGroup/SPIRV-Cross

```c++
#include <fstream>
#include <vector>
#include <spirv_cross.hpp>

// 读取 SPIR-V 文件
std::ifstream file("shader.spv", std::ios::binary);
std::vector<uint32_t> spirv_data((std::istreambuf_iterator<char>(file)), std::istreambuf_iterator<char>());

// 创建 SPIRV-Cross 编译器对象
spirv_cross::Compiler compiler(spirv_data);

// 获取输入和输出变量信息
auto input_vars = compiler.get_active_interface_variables(spirv_cross::ShaderStage::Vertex, spirv_cross::ActiveVariableInfo::StorageClassInput);
auto output_vars = compiler.get_active_interface_variables(spirv_cross::ShaderStage::Vertex, spirv_cross::ActiveVariableInfo::StorageClassOutput);

// 将 SPIR-V 转换为 GLSL
spirv_cross::CompilerGLSL glsl_compiler(compiler.get_parsed_ir());
glsl_compiler.set_common_options(default_glsl_options);
std::string glsl_source = glsl_compiler.compile();

```