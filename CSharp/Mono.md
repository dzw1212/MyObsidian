*http://nilssondev.com/mono-guide/book/introduction/about-mono.html#about-mono*

`Mono`是微软的`.NET Framework`的一个开源实现，最初的目标是实现.NET Framework的跨平台，这部分工作已由`.NET Core`接任，目前Mono一般用于游戏引擎中实现脚本系统；

# 版本

Mono分为`经典Mono`和`.NET Core Mono`，经典Mono只支持到`C#7`或者说`.NET Framework 4.7.2`，.NET Core Mono则支持最新版本的C#；
经典Mono没有被集成到.NET中，但其更简单，也更适合游戏引擎；

## 为什么不用最新的Mono

最新的.NET Core Mono**不支持汇编重载**；
游戏引擎对脚本系统的一个要求是——能够在修改脚本后热重载该脚本，而不同重新启动整个编辑器；为了完成这个要求，需要先卸载旧的程序集，再加载新的程序集；
目前，.NET Core Mono尚不支持重载程序集，而经典Mono可以，因此选择经典Mono来构建脚本系统；

# 构建

## Windows平台

### 构建Mono运行时库

下载源代码，使用`msvc/mono.sln`进行构建（建议配置`Debug x64`和`Release x64`）；
*https://github.com/mono/mono*

所需要的库文件都放置在`msvc/build/sgen/{platform}/`目录下，所需要的文件有：
- `lib`目录下：
	eglib.lib
	libgcmonosgen.lib
	libmini-sgen.lib
	libmonoruntime-sgen.lib
	libmono-static-gen.lib
	libmonoutils.lib
	mono-2.0-sgen.lib
	MonoPosixHelper.lib
- `bin`目录下：
	mono-2.0-sgen.dll
	MonoPosixHelper.dll

### 复制库文件

下载并安装Mono：*https://www.mono-project.com/download/stable/*

推荐使用`C:\Program Files\Mono\lib\mono\4.5\`，将该目录及其下所有文件复制到游戏引擎的某个目录下，等待后续链接；

# 嵌入引擎

## 初始化Runtime

```c++
#include "mono/jit/jit.h"
#include "mono/metadata/assembly.h"

void InitMono()
{
	//指定mono lib库的位置
	mono_set_assemblies_path("mono/lib/4.5");
}
```

如果未显式指定，Mono会尝试根据环境变量`MONO_PATH`来定位`mscorlib.dll`；

如果未找到lib库，则会报错*The assembly mscorlib.dll was not found or could not be loaded.*

```c++
void InitMono()
{
	mono_set_assemblies_path("mono/lib");

	//mono_jit_init_version可以指定版本，但一般让mono自己选择
	//MyScriptRuntime为自定义的Mono Runtime的名称
	MonoDomain *rootDomain = mono_jit_init("MyScriptRuntime");
	if (!rootDomain)
		return;

	//需要手动管理该指针
	s_RootDomain = rootDomain;
}
```

## 创建App Domain

```c++
//MyAppDomain为自定义名称
//第二个参数允许传递一个配置文件的路径
//获得的指针同样需要手动管理
s_AppDomain = mono_domain_create_appdomain("MyAppDomain", nullptr);

//将s_AppDomain设为当前应用域，第二个参数表示是否强制
mono_domain_set(s_AppDomain, true);
```

## 加载C# Assemblies

C#的程序集可以是一个`dll`或`exe`文件；

通常在一个游戏引擎中有两个需要加载的dll，一个是游戏代码DLL，一个是由引擎提供的C# DLL，游戏代码会链接到这个DLL；
因此需要一个通用函数，能够加载任何DLL，返回一个`MonoAssembly`指针；

Step1、将DLL文件读取进一个字节数组中：
```c++
static char* ReadBytes(const std::string& filepath, uint32_t* outSize)
{
	std::ifstream stream(filepath, std::ios::binary | std::ios::ate);

	if (!stream)
	{
		// Failed to open the file
		return nullptr;
	}

	std::streampos end = stream.tellg();
	stream.seekg(0, std::ios::beg);
	uint32_t size = end - stream.tellg();

	if (size == 0)
	{
		// File is empty
		return nullptr;
	}

	char* buffer = new char[size];
	stream.read((char*)buffer, size);
	stream.close();

	*outSize = size;
	return buffer;
}
```

Step2、加载C#程序集；
```c++
static MonoAssembly* LoadCSharpAssembly(const std::string& assemblyPath)
{
	uint32_t fileSize = 0;
	char* fileData = ReadBytes(assemblyPath, &fileSize);

	// NOTE: We can't use this image for anything other than loading the assembly because this image doesn't have a reference to the assembly
	MonoImageOpenStatus status;
	//param1：数据指针；
	//param2：数据长度；
	//param3：是否复制数据，若为true则mono会复制数据到一个内部缓冲区
	//param4：一个枚举指针，用来判断数据传递是否成功
	//param5：若为true，则mono将以反射模式加载，此时可以检查类型但不能运行代码
	MonoImage* image = mono_image_open_from_data_full(fileData, fileSize, true, &status, false);

	if (status != MONO_IMAGE_OK)
	{
		const char* errorMessage = mono_image_strerror(status);
		// Log some error message using the errorMessage data
		return nullptr;
	}

	//param1：mono加载的image
	//param2：mono在打印错误时可以使用的名字
	//param3：状态枚举指针
	//param4：是否启用反射模式
	MonoAssembly* assembly = mono_assembly_load_from_full(image, assemblyPath.c_str(), &status, 0);

	//使用完后释放mono image
	mono_image_close(image);

	// Don't forget to free the file data
	delete[] fileData;

	return assembly;
}
```

## 测试Assembly加载

为了测试程序集是否正确加载，可以遍历所有定义在程序集中的类的类型，查看有哪些类、结构体、枚举；
```c++
static void PrintAssemblyTypes(MonoAssembly* assembly)
{
	MonoImage* image = mono_assembly_get_image(assembly);
	const MonoTableInfo* typeDefinitionsTable = mono_image_get_table_info(image, MONO_TABLE_TYPEDEF);
	int32_t numTypes = mono_table_info_get_rows(typeDefinitionsTable);

	for (int32_t i = 0; i < numTypes; i++)
	{
		uint32_t cols[MONO_TYPEDEF_SIZE];
		//填充第i行的数据
		//param1：需要迭代的表格
		//param2：想要读取的列
		//param3：用户分配的列数组
		//param4：用户分配的列数组的大小
		mono_metadata_decode_row(typeDefinitionsTable, i, cols, MONO_TYPEDEF_SIZE);

		//获取不同类型的数据
		const char* nameSpace = mono_metadata_string_heap(image, cols[MONO_TYPEDEF_NAMESPACE]);
		const char* name = mono_metadata_string_heap(image, cols[MONO_TYPEDEF_NAME]);

		std::cout << "---row: " << i << std::endl;
		std::cout << "---namespace: " << nameSpace << std::endl;
		std::cout << "---name: " <<name << std::endl;
	}
}
```

```c#
using System;

namespace DAZEL
{
	public class Main
	{
		public float FloatVar { get; set; }

		public Main()
		{
			Console.WriteLine("Main constructor");
		}

		public void PrintMessage()
		{
			Console.WriteLine("Hello World From C#");
		}

		public void PrintCustomMessage(string msg)
		{
			Console.WriteLine($"C# says: {msg}");
		}
	}
}

//---row: 0
//---namespace:
//---name: <Module>
//---row: 1
//---namespace: DAZEL
//---name: Main
```

打印出来的第一个类型名称是`<Module>`，这是一个由C#编译器提供的类型，所有的dll和exe都带有这个类型，一个Assembly至少带有一个Module；

## 获取C#类的引用

为了能够从C++代码中调用C#类的方法并访问其属性，需要先获取C#类的引用，并在C++中创建一个对应的实例；

```c++
MonoClass* GetClassInAssembly(MonoAssembly* assembly, const char* namespaceName, const char* className)
{
    MonoImage* image = mono_assembly_get_image(assembly);
    MonoClass* klass = mono_class_from_name(image, namespaceName, className);

    if (klass == nullptr)
    {
        // Log error here
        return nullptr;
    }

    return klass;
}

// ...

MonoClass* testingClass = GetClassInAssembly(appAssembly, "", "CSharpTesting");
```

## 实例化C#类

```c++
// Get a reference to the class we want to instantiate
MonoClass* testingClass = GetClassInAssembly(appAssembly, "", "CSharpTesting");

// Allocate an instance of our class
MonoObject* classInstance = mono_object_new(s_AppDomain, testingClass);

if (classInstance == nullptr)
{
    // Log error here and abort
}

// Call the parameterless (default) constructor
mono_runtime_object_init(classInstance);
```