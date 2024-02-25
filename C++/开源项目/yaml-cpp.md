[[yaml语法]]

`Node`是`yaml-cpp`中的核心概念，是最重要的数据结构，用于存储解析后的`yaml`信息；

# Node的类型

`Node`的类型有：
```c++
struct NodeType {
	enum value 
	{ 
	    Undefined, 
	    Null, 
	    Scalar, //字面量
	    Sequence, //数组
	    Map //对象
	};
};
```

查询`Node`的类型：
```c++
node.type();
node.IsSequence();
```

`yaml`数组与对象的使用方法类似于`std::vector`和`std::map`；
```c++
//Sequence
YAML::Node primes = YAML::Load("[2,3,4,5,6]");
primes.push_back(7);
for (std::size_t i = 0; i < primes.size(); ++i)
	primes[i].as<int>();

for (YAML::const_iterator it = primes.begin(); it != primes.end(); ++it)
	it->as<int>();

//Map
YAML::Node lineup = YAML::Load("{1B: Prince Fielder, 2B: Rickie Weeks, LF: Ryan Braun}");
for (YAML::const_iterator it = lineup.begin(); it != lineup.end(); ++it
{
  it->first.as<std::string>();
  it->second.as<std::string>();
}
lineup["RF"] = "Corey Hart";
lineup["C"] = "Jonathan Lucroy";
assert(lineup.size() == 5);
```

# 构建Node

```c++
YAML::Node node;  // starts out as null
node["key"] = "value";  // it now is a map node
node["seq"].push_back("first element");  // node["seq"] automatically becomes a sequence
node["seq"].push_back("second element");

node["mirror"] = node["seq"][0];  // this creates an alias
node["seq"][0] = "1st element";  // this also changes node["mirror"]
node["mirror"] = "element #1";  // and this changes node["seq"][0] - they're really the "same" node

node["self"] = node;  // you can even create self-aliases
node[node["mirror"]] = node["seq"];  // and strange loops :)
```

# 数组转为对象

```c++
YAML::Node node  = YAML::Load("[1, 2, 3]");
node[1] = 5;  // still a sequence, [1, 5, 3]
node.push_back(-3) // still a sequence, [1, 5, 3, -3]
node["key"] = "value"; // now it's a map! {0: 1, 1: 5, 2: 3, 3: -3, key: value}

YAML::Node node = YAML::Load("[1, 2, 3]");
node[3] = 4; // still a sequence, [1, 2, 3, 4]
node[10] = 10;  // now it's a map! {0: 1, 1: 2, 2: 3, 3: 4, 10: 10}
```

# yaml类型与原生类型的转换

```c++
YAML::Node node = YAML::Load("{pi: 3.14159, [0, 1]: integers}");

// this needs the conversion from Node to double
double pi = node["pi"].as<double>();

// this needs the conversion from double to Node
node["e"] = 2.71828;

// this needs the conversion from Node to std::vector<int> (*not* the other way around!)
std::vector<int> v;
v.push_back(0);
v.push_back(1);
std::string str = node[v].as<std::string>();
```

假如是自定义的数据类型，与`yaml`类型的转换需要定义转换函数`YAML::convert<template T>`；
```c++
struct Vec3 { double x, y, z; };
```

```c++
namespace YAML {
	template<>
	struct convert<Vec3> 
	{
		static Node encode(const Vec3& rhs) 
		{
			Node node;
			node.push_back(rhs.x);
			node.push_back(rhs.y);
			node.push_back(rhs.z);
			return node;
		}
		
		static bool decode(const Node& node, Vec3& rhs) 
		{
			if (!node.IsSequence() || node.size() != 3) 
			{
				return false;
			}
		
			rhs.x = node[0].as<double>();
			rhs.y = node[1].as<double>();
			rhs.z = node[2].as<double>();
			return true;
		}
	};
}

//之后便支持Sequence与Vec3的互转
YAML::Node node = YAML::Load("start: [1, 3, 0]");
Vec3 v = node["start"].as<Vec3>();
node["end"] = Vec3(2, -1, 0);
```

# 发送yaml

发送`yaml`需要使用`YAML::Emitter`，其使用方式类似于`std::ostream`，并且可以通过`c_str()`函数获取到输出的内容；
```c++
#include "yaml-cpp/yaml.h"

int main()
{
   YAML::Emitter out;
   out << "Hello, World!";
   
   std::cout << "Here's the output YAML:\n" << out.c_str(); // prints "Hello, World!"
   return 0;
}
```

## 发送Sequence和Map

`YAML::Emitter`的使用类似于状态机，在输出不同类型的数据时需要不同的状态；
```c++
//输出Sequence
YAML::Emitter out;
out << YAML::BeginSeq;
out << "eggs";
out << "bread";
out << "milk";
out << YAML::EndSeq;

//产生数据：
 - eggs
 - bread
 - milk

//输出Map
YAML::Emitter out;
out << YAML::BeginMap;
out << YAML::Key << "name";
out << YAML::Value << "Ryan Braun";
out << YAML::Key << "position";
out << YAML::Value << "LF";
out << YAML::EndMap;

//产生数据：
name: Ryan Braun
position: LF

//Sequence和Map混合
YAML::Emitter out;
out << YAML::BeginMap;
out << YAML::Key << "name";
out << YAML::Value << "Barack Obama";
out << YAML::Key << "children";
out << YAML::Value << YAML::BeginSeq << "Sasha" << "Malia" << YAML::EndSeq;
out << YAML::EndMap;

//产生数据：
name: Barack Obama
children:
  - Sasha
  - Malia
```

## 使用控制符

### YAML::Literal
```c++
YAML::Emitter out;
out << YAML::Literal << "A\n B\n  C";

A
 B
  C
```

### YAML::Flow
```c++
YAML::Emitter out;
out << YAML::Flow;
out << YAML::BeginSeq << 2 << 3 << 5 << 7 << 11 << YAML::EndSeq;

[2, 3, 5, 7, 11]
```

### YAML::Comment
```c++
YAML::Emitter out;
out << YAML::BeginMap;
out << YAML::Key << "method";
out << YAML::Value << "least squares";
out << YAML::Comment("should we change this method?");
out << YAML::EndMap;

method: least squares  # should we change this method?
```

### YAML::Alias
```c++
YAML::Emitter out;
out << YAML::BeginSeq;
out << YAML::Anchor("fred");
out << YAML::BeginMap;
out << YAML::Key << "name" << YAML::Value << "Fred";
out << YAML::Key << "age" << YAML::Value << "42";
out << YAML::EndMap;
out << YAML::Alias("fred");
out << YAML::EndSeq;

- &fred
  name: Fred
  age: 42
- *fred
```

## STL类型的<<重载

yaml-cpp提供了`std::vector`，`std::list`和`std::map`的`operator <<`重载；
```c++
std::vector <int> squares;
squares.push_back(1);
squares.push_back(4);
squares.push_back(9);
squares.push_back(16);

std::map <std::string, int> ages;
ages["Daniel"] = 26;
ages["Jesse"] = 24;

YAML::Emitter out;
out << YAML::BeginSeq;
out << YAML::Flow << squares;
out << ages;
out << YAML::EndSeq;

- [1, 4, 9, 16]
-
  Daniel: 26
  Jesse: 24
```

## 重载<<操作符

```c++
struct Vec3 { int x; int y; int z; };
YAML::Emitter& operator << (YAML::Emitter& out, const Vec3& v) {
	out << YAML::Flow;
	out << YAML::BeginSeq << v.x << v.y << v.z << YAML::EndSeq;
	return out;
}
```

## 设置输出编码格式

```c++
emitter.SetOutputCharset(YAML::EscapeNonAscii);
```

## 错误排查

```c++
YAML::Emitter out;
assert(out.good());
out << YAML::Key;
assert(!out.good());
std::cout << "Emitter error: " << out.GetLastError() << "\n";
```
