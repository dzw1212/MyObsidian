`protobuf`（`Protocol Buffers`）是由Google开发的一种数据序列化协议。它用于序列化结构化数据，类似于`XML`或`JSON`，但是比它们更小、更快、更简单。`protobuf`最初设计用于内部通信协议和文件格式，但现在被广泛用于服务间通信以及数据存储。使用`protobuf`，首先需要定义数据结构（在`.proto`文件中定义），然后使用`protobuf`编译器生成源代码，用于序列化和反序列化数据。支持多种编程语言，包括C++、Java、Python等。

*https://github.com/protocolbuffers/protobuf*

*https://protobuf.dev/getting-started/cpptutorial/*

# protobuf编译器与运行时

要安装`protobuf`，需要安装`protobuf`编译器（用来编译`.proto`文件）和选择对应语言的`protobuf runtime`；

# proto2和proto3

`proto2`和`proto3`是`protobuf`的两个主要版本，它们在语法和功能上有一些关键的区别：

1. **字段可选性**
- `proto2` 提供了三种字段类型：`required`、`optional`、`repeated`。`required`字段必须在消息中被设置，否则消息就会被认为是不完整的。`optional`字段可以没有，如果没有设置，就会有一个默认值。`repeated`字段用于表示数组或列表。
- `proto3` 移除了`required`和`optional`关键字，所有字段默认是可选的，但不会被自动设置为默认值。如果字段未被设置，它在序列化的消息中就不会占据任何空间。`repeated`字段保留，用于列表或数组。

2. **默认值**
- `proto2` 允许你为`optional`字段指定默认值。
- `proto3` 中，字段没有被显式设置时，将总是被初始化为其类型的默认值（例如，数字类型为0，字符串为空字符串等），并且不支持自定义默认值。

3. **语法简化**
- `proto3` 的设计目标之一是简化语法，使其更易于使用。例如，移除了`required`字段，减少了消息定义中可能导致向后兼容问题的特性。

4. **JSON映射**
- `proto3` 引入了更自然的JSON映射，使得`protobuf`消息和JSON之间的转换更加直接和自然。
# .proto文件

`.proto`文件为每个需要序列化的结构体添加`message`修饰，然后为每个`message`中的字段指定名称和类型；

```c
syntax = "proto2";

package tutorial;

message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    PHONE_TYPE_UNSPECIFIED = 0;
    PHONE_TYPE_MOBILE = 1;
    PHONE_TYPE_HOME = 2;
    PHONE_TYPE_WORK = 3;
  }

  message PhoneNumber {
    optional string number = 1;
    optional PhoneType type = 2 [default = PHONE_TYPE_HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

## 修饰符

- `optional`：这个字段在二进制包中可以有也可以没有，如果没有的话就使用默认值，对于简单类型用户可以自定义默认值（如示例中的`PhoneType`的四个值），否则默认数字类型为0、字符串类型为空字符串、布尔值为`false`；

- `repeated`：该字段可以重复任意次数（包括0次），视作动态大小的数组；

- `required`：二进制包中必须有该字段的值，否则会认为读取二进制包失败；需要谨慎使用该修饰符，`proto3`中已移除该修饰符；

## 编号

编号为该字段在二进制包中的唯一编号，编号1-15只需要一个字节进行编码，因此常用于使用频率高的字段，编号16及以上的留给不太常用的字段；

## 编译

假如文件名为`addressbook.proto`，使用protobuf编译器编译该文件：

```cpp
protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/addressbook.proto
```

会在目标目录下生成`addressbook.pb.h`和`addressbook.pb.cc`文件，分别对应类的头文件和源文件；

编译器会为`.proto`文件中的每个`message`体生成一个类，为类中的每个字段生成访问器；

### 字段方法

可以看到，`protobuf`为每个字段生成的方法有：
1. 是否有该字段，`has_XXX()`；
2. 清除该字段，`clear_XXX()`；
3. 获取该字段，`XXX()`；
4. 设置该字段，`set_XXX()`；
5. 获取该字段的指针，`mutable_XXX()`，仅限于复杂类型，如字符串（如果该字段未设置，`mutale_XXX`也会返回一个空字符串的指针而不是`nullptr`）；

---
对于`repeated`字段的特有方法：
6. 获取长度，`XXX_size()`；
7. 添加新成员，`add_XXX()`；
8. 通过索引获取某个成员，`XXX(int index)`，`mutable_XXX(int index)`；

```cpp
  // name
  inline bool has_name() const;
  inline void clear_name();
  inline const ::std::string& name() const;
  inline void set_name(const ::std::string& value);
  inline void set_name(const char* value);
  inline ::std::string* mutable_name();

  // id
  inline bool has_id() const;
  inline void clear_id();
  inline int32_t id() const;
  inline void set_id(int32_t value);

  // email
  inline bool has_email() const;
  inline void clear_email();
  inline const ::std::string& email() const;
  inline void set_email(const ::std::string& value);
  inline void set_email(const char* value);
  inline ::std::string* mutable_email();

  // phones
  inline int phones_size() const;
  inline void clear_phones();
  inline const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >& phones() const;
  inline ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >* mutable_phones();
  inline const ::tutorial::Person_PhoneNumber& phones(int index) const;
  inline ::tutorial::Person_PhoneNumber* mutable_phones(int index);
  inline ::tutorial::Person_PhoneNumber* add_phones();
```

对于例子中的枚举，可以用以下方式访问：
	`Person::PHONE_TYPE_UNSPECIFIED`
	`Person::PHONE_TYPE_MOBILE`
	`Person::PHONE_TYPE_HOME`
	`Person::PHONE_TYPE_WORK`

对于例子中的嵌套类`PhoneNumber`，可以用以下方式访问（实际的名称是`Person_PhoneNumber`,前向声明时需要使用实际名称）：
	`Person::PhoneNumber`

### Message方法

```cpp
//检查是否已设置所有必填字段，即required字段
bool IsInitialized() const;

//返回可读消息，用于调试
string DebugString() const;

//用给定的值覆盖此消息
void CopyFrom(const Person& from);

//清空Message
void Clear();
```

# 序列化

```cpp
//序列化message到给定字符串中（是不可读的二进制数据而非文本，字符串只是个容器）
bool SerializeToString(std::string* output) const;

//从给定字符串中解析出message
bool ParseFromString(const std::string& data);

//序列化message到给定的ostream
bool SerializeToOstream(std::ostream* output) const;

//从给定istream中解析出message
bool ParseFromIstream(std::istream* input);
```

## 读取与写入Message

下面是一个程序，他从文件中读取`AddressBook`，向其中添加一个新的`Person`，然后将修改后的`AddressBook`写回到文件中；

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include "addressbook.pb.h"

int main()
{
	//验证确保没有意外链接到与编译的不兼容的库版本
	//不必要，但是建议
	GOOGLE_PROTOBUF_VERIFY_VERSION;

	//命名空间名称与proto文件中对应，"package tutorial"
	tutorial::AddressBook address_book;
	const char* filepath = "./myAddressBook.blob";

	auto people = address_book.add_people();
	people->set_name("Jack");
	people->set_id(123);
	people->set_email("Jack@gmail.com");

	auto phone1 = people->add_phones();
	phone1->set_type(tutorial::Person::PHONE_TYPE_HOME);
	phone1->set_number("123456");
	auto phone2 = people->add_phones();
	phone2->set_type(tutorial::Person::PHONE_TYPE_MOBILE);
	phone2->set_number("123456789");

	std::fstream output(filepath, std::ios::out | std::ios::trunc | std::ios::binary);
	if (!address_book.SerializeToOstream(&output))
		return -1;

	/*********************************************************/

	address_book.Clear();
	std::fstream input(filepath, std::ios::in | std::ios::binary);
	if (!input)
		return -1;
	if (!address_book.ParseFromIstream(&input))
		return -1;
	if (address_book.people_size() == 0)
		return -1;

	auto firstPeople = address_book.people(0);
	std::cout << firstPeople.name() << std::endl;

	//释放libprotobuf占用的资源
	//不强制，看需求调用
	google::protobuf::ShutdownProtobufLibrary();

	return 0;
}

```

# 使用步骤

0. 获取`protobuf`源代码，编译生成库文件，添加到项目中（狗屎中的狗屎）；

	如果想自己编译，需要先`git submodule update --init --recursive`获取所有子模块（大概率会遇到网络问题），然后使用`CMake`生成vs项目，在vs中编译；

	或者直接用`vcpkg`下载，但需要注意的是，`vcpkg`下载的lib版本需要和`protobuf`编译器的版本一致，否则会编译不通过；

	属性->链接器->常规->附加库目录：填入`libprotobuf.lib`所在目录；
	属性->链接器->输入->附加依赖项：填入`libprotobuf.lib`；
	属性->C/C++->常规->附加包含目录：填入`include`所在目录；
	将`libprotobuf.dll`置于项目可执行文件同目录；

2. 编写`.proto`文件；
3. 编译刚才编写的`.proto`文件，生成对应的`.pb.h`和`.pb.cc`文件；
4. 将生成的文件添加到项目中使用；

# 处理版本问题

`protobuf`处理数据版本不一致的问题主要依赖于其设计中的几个关键特性：

1. 字段编号（`Field Numbers`）：在`protobuf`中，每个字段都通过唯一的编号（而不是字段名）来标识，一旦某个字段编号被分配给了某个字段，即使该字段被删除，这个编号也不应该再被用于其他任何新字段；这意味着字段的名字可以改变，但编号保持不变，确保了向后兼容性；

2. 可选（`Optional`）和重复（`Repeated`）字段：字段可以被标记为可选或重复，这提供了灵活性。如果一个新版本的类中添加了字段，旧版本的代码在反序列化时会忽略这些新字段，因为它们在旧版本中不存在。同样，如果删除了某些字段，新版本的代码在处理旧数据时会忽略这些不存在的字段；

3. 默认值（`Default`）：未设置的字段将被赋予其类型的默认值，这意味着即使数据中没有某个字段的信息，代码也能正常运行，因为它会使用该字段类型的默认值；

当旧版本的代码读取包含新版本字段的数据时，这些新字段会被忽略，因为旧版本的代码不认识这些新字段。这保证了新数据能够被旧代码安全处理；

当新版本的代码读取旧版本的数据时，任何新添加的字段都将被设置为其默认值（如果有的话），因为旧数据中不包含这些字段。同时，任何在新版本中删除的字段，在旧数据中的值也会被忽略；

# 踩坑

打印字符串时，使用`printf`没问题，但使用`std::cout`就会出现错误的结果，非字符串不会有这种问题；