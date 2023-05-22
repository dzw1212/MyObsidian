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

# 使用

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
// 根据类的名称获取C#类的引用
MonoClass* testingClass = GetClassInAssembly(appAssembly, "", "CSharpTesting");

// 分配实例
MonoObject* classInstance = mono_object_new(s_AppDomain, testingClass);

if (classInstance == nullptr)
{
    // Log error here and abort
}

// 尝试调用默认的类的构造函数
// 如果找不到无参数的构造函数，则会触发assert
mono_runtime_object_init(classInstance);
```

## 从C++中调用C#函数

### 两种调用C#函数的方法

1、`mono_runtime_invoke`
相比Unmanaged Method Thunks，更慢，更安全，更灵活；
可以用任何参数调用任何方法，mono_runtime_invoke会对传递的对象以及参数进行检查验证；
2、`Unmanaged Method Thunks`
相比mono_runtime_invoke，允许以更少的开销调用C#方法；

如果一个C#函数每秒钟要调用多次，并且在编译时知道该方法的签名，则应该使用Unmanaged Method Thunks；否则应该使用mono_runtime_invoke；

### 检索与调用

```c++
MonoClass* instanceClass = mono_object_get_class(objectInstance);

// 根据函数名称来获取C#函数的引用
//param1：类的实例
//param2：函数名称
//param3：函数参数的个数，若为-1，则Mono会返回找到的函数的第一个版本
//        如果有多个版本的函数具有相同的参数个数，则需要其他方法来获取函数引用
MonoMethod* method = mono_class_get_method_from_name(instanceClass, "PrintFloatVar", 0);

if (method == nullptr)
{
	//未找到对应的函数
	return;
}

//调用C#函数
MonoObject* exception = nullptr;
//param1：函数引用
//param2：类对象实例
//param3：想要传递的任何参数的数组指针
//param4：异常变量的指针
mono_runtime_invoke(method, objectInstance, nullptr, &exception);
```

### 传递参数

```c++
MonoClass* instanceClass = mono_object_get_class(objectInstance);
MonoMethod* method = mono_class_get_method_from_name(instanceClass, "IncrementFloatVar", 1);
if (!method)
	return;


float value = 0.5f;
MonoObject* exception = nullptr;

//方法一
void* param = &value;
mono_runtime_invoke(method, objectInstance, &param, &exception);

//方法二
void* params[] =
{
	&value
};

mono_runtime_invoke(method, objectInstance, params, &exception);
```

如果是要传递的参数是`string`，则需要首先用`mono_string_new`将参数字符串转为`MonoString`；

```c++
std::string strMsg("Go away");
void* param = mono_string_new(s_ScriptEngineData->pAppDomain, strMsg.c_str());
mono_runtime_invoke(method2, classInstance, &param, nullptr);
```

### 访问C#字段与属性
[[CSharp/概念#字段与属性]]

样本代码：
```C#
using System;

public class CSharpTesting
{
    public float MyPublicFloatVar = 5.0f;

    private string m_Name = "Hello";
    public string Name
    {
        get => m_Name;
        set
        {
            m_Name = value;
            MyPublicFloatVar += 5.0f;
        }
    }

    public void PrintFloatVar()
    {
        Console.WriteLine("MyPublicFloatVar = {0:F}", MyPublicFloatVar);
    }

    private void IncrementFloatVar(float value)
    {
        MyPublicFloatVar += value;
    }

}
```

```C++
MonoObject* testingInstance = InstantiateClass("", "CSharpTesting");
MonoClass* testingClass = mono_object_get_class(testingInstance);

// Get a reference to the public field called "MyPublicFloatVar"
MonoClassField* floatField = mono_class_get_field_from_name(testingClass, "MyPublicFloatVar");

// Get a reference to the private field called "m_Name"
MonoClassField* nameField = mono_class_get_field_from_name(testingClass, "m_Name");

// Get a reference to the public property called "Name"
MonoProperty* nameProperty = mono_class_get_property_from_name(testingClass, "Name");

// Do something
```

### 检查可访问性

Mono不关心类、方法、字段或属性的可访问性，比如Mono允许用户设置一个私有字段的值，但是出于对脚本逻辑与安全性的考虑，用户应该尊重字段或属性的可访问性；

```C++
enum class Accessibility : uint8_t
{
	None		= 0,
	Private		= (1 << 0),
	Internal	= (1 << 1),
	Protected	= (1 << 2),
	Public		= (1 << 3),
};
```

```C++
// Gets the accessibility level of the given field
static uint8_t GetFieldAccessibility(MonoClassField* field)
{
	uint8_t accessibility = (uint8_t)Accessibility::None;
	uint32_t accessFlag = mono_field_get_flags(field) & MONO_FIELD_ATTR_FIELD_ACCESS_MASK;

	switch (accessFlag)
	{
	case MONO_FIELD_ATTR_PRIVATE:
		{
			accessibility = (uint8_t)Accessibility::Private;
			break;
		}
	case MONO_FIELD_ATTR_FAM_AND_ASSEM:
		{
			accessibility |= (uint8_t)Accessibility::Protected;
			accessibility |= (uint8_t)Accessibility::Internal;
			break;
		}
	case MONO_FIELD_ATTR_ASSEMBLY:
		{
			accessibility = (uint8_t)Accessibility::Internal;
			break;
		}
	case MONO_FIELD_ATTR_FAMILY:
		{
			accessibility = (uint8_t)Accessibility::Protected;
			break;
		}
	case MONO_FIELD_ATTR_FAM_OR_ASSEM:
		{
			accessibility |= (uint8_t)Accessibility::Private;
			accessibility |= (uint8_t)Accessibility::Protected;
			break;
		}
	case MONO_FIELD_ATTR_PUBLIC:
		{
			accessibility = (uint8_t)Accessibility::Public;
			break;
		}
	}

	return accessibility;
}

// Gets the accessibility level of the given property
static uint8_t GetPropertyAccessbility(MonoProperty* property)
{
	uint8_t accessibility = (uint8_t)Accessibility::None;

	// Get a reference to the property's getter method
	MonoMethod* propertyGetter = mono_property_get_get_method(property);
	if (propertyGetter != nullptr)
	{
		// Extract the access flags from the getters flags
		uint32_t accessFlag = mono_method_get_flags(propertyGetter, nullptr) & MONO_METHOD_ATTR_ACCESS_MASK;

		switch (accessFlag)
		{
		case MONO_FIELD_ATTR_PRIVATE:
			{
				accessibility = (uint8_t)Accessibility::Private;
				break;
			}
		case MONO_FIELD_ATTR_FAM_AND_ASSEM:
			{
				accessibility |= (uint8_t)Accessibility::Protected;
				accessibility |= (uint8_t)Accessibility::Internal;
				break;
			}
		case MONO_FIELD_ATTR_ASSEMBLY:
			{
				accessibility = (uint8_t)Accessibility::Internal;
				break;
			}
		case MONO_FIELD_ATTR_FAMILY:
			{
				accessibility = (uint8_t)Accessibility::Protected;
				break;
			}
		case MONO_FIELD_ATTR_FAM_OR_ASSEM:
			{
				accessibility |= (uint8_t)Accessibility::Private;
				accessibility |= (uint8_t)Accessibility::Protected;
				break;
			}
		case MONO_FIELD_ATTR_PUBLIC:
			{
				accessibility = (uint8_t)Accessibility::Public;
				break;
			}
		}
	}

	// Get a reference to the property's setter method
	MonoMethod* propertySetter = mono_property_get_set_method(property);
	if (propertySetter != nullptr)
	{
		// Extract the access flags from the setters flags
		uint32_t accessFlag = mono_method_get_flags(propertySetter, nullptr) & MONO_METHOD_ATTR_ACCESS_MASK;
		if (accessFlag != MONO_FIELD_ATTR_PUBLIC)
			accessibility = (uint8_t)Accessibility::Private;
	}
	else
	{
		accessibility = (uint8_t)Accessibility::Private;
	}

	return accessibility;
}
```

```C++
MonoObject* testingInstance = InstantiateClass("", "CSharpTesting");
MonoClass* testingClass = mono_object_get_class(testingInstance);

// Get a reference to the public field called "MyPublicFloatVar"
MonoClassField* floatField = mono_class_get_field_from_name(testingClass, "MyPublicFloatVar");
uint8_t floatFieldAccessibility = GetFieldAccessibility(floatField);

if (floatFieldAccessibility & (uint8_t)Accessibility::Public)
{
    // We can safely write a value to this
}

// Get a reference to the private field called "m_Name"
MonoClassField* nameField = mono_class_get_field_from_name(testingClass, "m_Name");
uint8_t nameFieldAccessibility = GetFieldAccessibility(nameField);

if (nameFieldAccessibility & (uint8_t)Accessibility::Private)
{
    // We shouldn't write to this field
}

// Get a reference to the public property called "Name"
MonoProperty* nameProperty = mono_class_get_property_from_name(testingClass, "Name");
uint8_t namePropertyAccessibility = GetPropertyAccessibility(nameProperty);

if (namePropertyAccessibility & (uint8_t)Accessibility::Public)
{
    // We can safely write a value to the field using this property
}
```

### 设置和获取值

```C++
static bool CheckMonoError(MonoError& error)
{
	bool hasError = !mono_error_ok(&error);
	if (hasError)
	{
		unsigned short errorCode = mono_error_get_error_code(&error);
		const char* errorMessage = mono_error_get_message(&error);
		std::cout << "Mono Error!" << std::endl;
		std::cout << "Error Code: "<< errorCode << std::endl;
		std::cout << "Error Message: " << errorMessage << std::endl;
		mono_error_cleanup(&error);
	}
	return hasError;
}

static std::string MonoStringToUTF8(MonoString* monoString)
{
	if (monoString == nullptr || mono_string_length(monoString) == 0)
		return "";

	MonoError error;
	char* utf8 = mono_string_to_utf8_checked(monoString, &error);
	if (CheckMonoError(error))
		return "";
	std::string result(utf8);
	mono_free(utf8);
	return result;
}
```

```C++
MonoObject* testingInstance = InstantiateClass("", "CSharpTesting");
MonoClass* testingClass = mono_object_get_class(testingInstance);

MonoClassField* floatField = mono_class_get_field_from_name(testingClass, "MyPublicFloatVar");

// Get the value of MyPublicFloatVar from the testingInstance object
float value;
//param1：类的实例
//param2：字段
//param3：用来存储变量的指针
mono_field_get_value(testingInstance, floatField, &value);

// Increment value by 10 and assign it back to the variable
value += 10.0f;
mono_field_set_value(testingInstance, floatField, &value);

MonoProperty* nameProperty = mono_class_get_property_from_name(testingClass, "Name");

// Get the value of Name by invoking the getter method
// mono_property_get_value返回一个MonoObject指针
// 如果是一个字符串对象，可以直接强转为MonoString；如果是一个值对象，则需要拆箱
MonoString* nameValue = (MonoString*)mono_property_get_value(nameProperty, testingInstance, nullptr, nullptr);
std::string nameStr = MonoStringToUTF8(nameValue);

// Modify and assign the value back to the property by invoking the setter method
nameStr += ", World!";
//需要构造一个新的MonoString，不能直接修改获取到的
nameValue = mono_string_new(s_AppDomain, nameStr.c_str());
//不能直接传递值的指针，需要封装进一个void*的数组
//如果是MonoObject*或MonoString*对象，则可以简单的使用void**
mono_property_set_value(nameProperty, testingInstance, (void**)&nameValue, nullptr);
```

拆箱属性的值对象：
```C++
MonoProperty* floatProperty = mono_class_get_property_from_name(testingClass, "MyFloatProperty");

// Get the value of Name by invoking the getter method
MonoObject* floatValueObj = mono_property_get_value(floatProperty, testingInstance, nullptr, nullptr);
float floatValue = *(float*)mono_object_unbox(floatValueObj);

// Modify and assign the value back to the property by invoking the setter method
floatValue += 10.0f;
void* data[] = { &floatValue };
mono_property_set_value(nameProperty, testingInstance, data, nullptr);
```

## 从C#中调用C++函数

### Platform Invoke

### 内部调用

C++端：
```c++
static void CppFunction()
{
	std::cout << "Call CppFunction from C++" << std::endl;
}

//after mono_domain_set

//注意！！！
//函数名称的格式为{namespaceName.className::functionName}
mono_add_internal_call("DAZEL.Main::CppFunction", CppFunction);
```

C#端：
```C#
using System;
using System.Runtime.CompilerServices;

namespace DAZEL
{
	public class Main
	{
		public float FloatVar { get; set; }

		public Main()
		{
			Console.WriteLine("Main default constructor");
		}

		public void PrintMessage()
		{
			Console.WriteLine("Hello World From C#");

			CppFunction();
		}

		public void PrintCustomMessage(string msg)
		{
			Console.WriteLine($"C# says: {msg}");
		}

		[MethodImplAttribute(MethodImplOptions.InternalCall)]
		extern static void CppFunction();
	}
}
```


