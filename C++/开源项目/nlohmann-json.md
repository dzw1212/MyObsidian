*https://github.com/nlohmann/json*

# 读取不存在的字段名

## 直接通过[]访问

如果Json数据`non-const`的，则会插入一个新字段，初值为`null`，并返回对该初值的引用，大部分情况下赋值会触发`nlohmann::json::type_error`异常；

如果Json数据是`const`的，则会触发`nlohmann::json::out_of_range`异常；
## .at方法

触发`nlohmann::json::out_of_range`异常；
## .find方法

`if (data.find("field") == data.end())`；
## .value方法

如果找不到字段会提供一个初值：`data.value("field", 0.f)`；

# 复杂类型序列化

`get_to`方法能够完成复杂类型的序列化与反序列化，前提是定义了该复杂类型的`from_json`和`to_json`方法并且`nlohmann-json`能够找到；

一、可以将`from_json/to_json`方法定义在复杂类型相同的命名空间内；

```cpp
#include <nlohmann/json.hpp>
#include <string>

namespace my_namespace {

struct Address {
    std::string city;
    std::string street;
};

struct Person {
    std::string name;
    int age;
    Address address;
};

void from_json(const nlohmann::json& j, Address& a) {
    j.at("city").get_to(a.city);
    j.at("street").get_to(a.street);
}

void to_json(nlohmann::json& j, const Address& a) {
    j = nlohmann::json{{"city", a.city}, {"street", a.street}};
}

void from_json(const nlohmann::json& j, Person& p) {
    j.at("name").get_to(p.name);
    j.at("age").get_to(p.age);
    j.at("address").get_to(p.address);
}

void to_json(nlohmann::json& j, const Person& p) {
    j = nlohmann::json{{"name", p.name}, {"age", p.age}, {"address", p.address}};
}

} // namespace my_namespace
```

二、在`nolhmann`的命名空间内[[模板特化]]`adl-serializer`；（个人更推荐，相对来说更清晰）

这种做法使用了一种名为[[参数依赖查找 ADL]]的技术；

```cpp
#include <nlohmann/json.hpp>
// 特化 nlohmann::adl_serializer 以提供自定义序列化逻辑
namespace nlohmann {
template<>
struct adl_serializer<Address> 
{
	static void to_json(json& j, const Address& a) 
	{
		j = json{{"city", a.city}, {"street", a.street}};
	}
	static void from_json(const json& j, Address& a) 
	{
		j.at("city").get_to(a.city);
		j.at("street").get_to(a.street);
	}
};
template<>
struct adl_serializer<Person> 
{
	static void to_json(json& j, const Person& p) 
	{
		j = json{{"name", p.name}, {"age", p.age}, {"address", p.address}};
	}
	static void from_json(const json& j, Person& p) 
	{
		j.at("name").get_to(p.name);
		j.at("age").get_to(p.age);
		j.at("address").get_to(p.address);
	}
};
} // namespace nlohmann
```

# 美观打印

```cpp
#include <iostream>
#include <nlohmann/json.hpp>

int main() {
    nlohmann::json j = {
        {"name", "John Doe"},
        {"age", 30},
        {"cars", {"BMW", "Tesla"}}
    };

    // 打印美观的JSON，缩进4个空格
    std::cout << j.dump(4) << std::endl;

    return 0;
}
```

# 与文件交互

## 写入文件

```cpp
#include <nlohmann/json.hpp>
#include <fstream>

int main() {
    // 创建一个JSON对象
    nlohmann::json j;
    j["name"] = "John Doe";
    j["age"] = 30;
    j["is_student"] = false;

    // 打开一个文件流进行写入
    std::ofstream o("example.json");
    o << j.dump(4); // 使用4个空格进行美化
    o.close();

    return 0;
}
```

## 从文件中读取

```cpp
#include <nlohmann/json.hpp>
#include <fstream>
#include <iostream>

int main() {
    // 创建一个空的JSON对象
    nlohmann::json j;

    // 打开文件并读取到JSON对象中
    std::ifstream i("example.json");
    i >> j;
    i.close();

    // 输出读取的JSON对象
    std::cout << j.dump(4) << std::endl;

    return 0;
}
```
