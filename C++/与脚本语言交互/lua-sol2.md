
`sol2`是一个现代的cpp-lua库，它利用了C++11及以上版本的语言特性，提供了更安全、更易用的API；`sol2`设计用于与Lua 5.1、5.2、5.3和LuaJIT兼容。它通过智能指针和类型推导等现代C++特性，大大简化了C++与Lua之间的交互；

*https://github.com/ThePhD/sol2*
*https://sol2.readthedocs.io/en/latest/tutorial/all-the-things.html*

# 基础

## 打开一个lua_state

```cpp
sol::state luaState;
```

## 加载库

```cpp
luaState.open_libraries(sol::lib::base);
```

全部的库有：
```cpp
enum class lib : unsigned char {
	// print, assert, and other base functions
	base,
	// require and other package functions
	package,
	// coroutine functions and utilities
	coroutine,
	// string library
	string,
	// functionality from the OS
	os,
	// all things math
	math,
	// the table manipulator and observer functions
	table,
	// the debug library
	debug,
	// the bit library: different based on which you're using
	bit32,
	// input/output library
	io,
	// LuaJIT only
	ffi,
	// LuaJIT only
	jit,
	// library for handling utf8: new to Lua
	utf8,
	// do not use
	count
};
```

## 传参

```lua
function Add(a, b)
    print(a + b)
end
```

```cpp
luaState.script_file("test.lua");
auto func = luaState["Add"];
if (func.valid())
    func(10, 20);
```
## 设置/读取变量

设置变量：
```cpp
// integer types
lua.set("number", 24);
// floating point numbers
lua["number2"] = 24.5;
// string types
lua["important_string"] = "woof woof";
// is callable, therefore gets stored as a function that can be called
lua["a_function"] = []() { return 100; };
// make a table
lua["some_table"] = lua.create_table_with("value", 24);
```

读取变量（在赋值前，必须确保该字段存在，否则会抛出异常）：
```cpp
// implicit conversion
int number = lua["number"]; 
c_assert(number == 24);
// explicit get
auto number2 = lua.get<double>("number2");
c_assert(number2 == 24.5);
// strings too
std::string important_string = lua["important_string"];
c_assert(important_string == "woof woof");
// dig into a table
int value = lua["some_table"]["value"];
c_assert(value == 24);
// get a function
sol::function a_function = lua["a_function"];
int value_is_100 = a_function();
// convertible to std::function
std::function<int()> a_std_function = a_function;
int value_is_still_100 = a_std_function();
c_assert(value_is_100 == 100);
c_assert(value_is_still_100 == 100);
```

对于未知类型的变量，可以先获取变量类型：
```cpp
sol::object number_obj = lua.get<sol::object>("number");
// sol::type::number
sol::type t1 = number_obj.get_type();
c_assert(t1 == sol::type::number);

sol::object function_obj = lua["a_function"];
// sol::type::function
sol::type t2 = function_obj.get_type();
c_assert(t2 == sol::type::function);
bool is_it_really = function_obj.is<std::function<int()>>();
c_assert(is_it_really);

// will not contain data
sol::optional<int> check_for_me = lua["a_function"];
c_assert(check_for_me == sol::nullopt);
```

通过将变量设为`nullptr`或`sol::lua_nil`来清除该变量；
```cpp
lua["number"] = nullptr;
lua.set("number", sol::lua_nil);
```

尝试获取变量，若不存在则返回默认值；
```cpp
lua.script("exists = 250");

int first_try = lua.get_or("exists", 322);
c_assert(first_try == 250);

lua.set("exists", sol::lua_nil);
int second_try = lua.get_or("exists", 322);
c_assert(second_try == 322);
```
## 表格 Table

`sol::state`也属于`sol::table`，对于表格的所以操作同样适用与`sol::state`；

```cpp
lua.script(R"(
	abc = { [0] = 24 }
	def = { 
		ghi = { 
			bark = 50, 
			woof = abc 
		} 
	}
)");

sol::table abc = lua["abc"];
sol::table def = lua["def"];
sol::table ghi = lua["def"]["ghi"];

int bark1 = def["ghi"]["bark"];
int bark2 = lua["def"]["ghi"]["bark"]; //注意，多层[]是一种危险的做法
c_assert(bark1 == 50);
c_assert(bark2 == 50);

int abcval1 = abc[0];
int abcval2 = ghi["woof"][0];
c_assert(abcval1 == 24);
c_assert(abcval2 == 24);
```

### 创建table

- `create_named_table`：将创建的表放入lua的全局命名空间并为其命名，可以直接通过lua脚本访问这个表；
- `create_table_with`：这种方法不会将创建的表放入lua的全局命名空间，适合将其作为参数在cpp和lua之间传递；

```cpp
lua["abc_sol2"] = lua.create_table_with(
	0, 24
);

sol::table inner_table = lua.create_table_with(
	"bark", 50,
	"woof", lua["abc_sol2"]
);
lua.create_named_table("def_sol2",
	"ghi", inner_table
);
```

## 函数 Function

lua的函数：
```cpp
lua.script("function f (a, b, c, d) return 1 end");
lua.script("function g (a, b) return a + b end");

// sol::function is often easier: 
// takes a variable number/types of arguments...
sol::function fx = lua["f"];
// fixed signature std::function<...>
// can be used to tie a sol::function down
std::function<int(int, double, int, std::string)> stdfx = fx;

int is_one = stdfx(1, 34.5, 3, "bark");
c_assert(is_one == 1);
int is_also_one = fx(1, "boop", 3, "bark");
c_assert(is_also_one == 1);

// call through operator[]
int is_three = lua["g"](1, 2);
c_assert(is_three == 3);
double is_4_8 = lua["g"](2.4, 2.4);
c_assert(is_4_8 == 4.8);
```

### 结构体成员的getter/setter

cpp的函数：
```cpp
void some_function() {
	std::cout << "some function!" << std::endl;
}

void some_other_function() {
	std::cout << "some other function!" << std::endl;
}

struct some_class {
	int variable = 30;

	double member_function() {
		return 24.5;
	}
};
```

下面的代码中，使用`m1`、`v1`这种方法的绑定的函数，在调用时需要指定一个some_class的实例，默认为第一个参数；而`m2`、`v2`在绑定时就指定了一个实例，调用时不需要再指定实例；
```cpp
// binds usertype data
lua.set("sc", some_class()); //sc为结构体some_class的一个实例

// binds a plain function
lua["f1"] = some_function;
lua.set_function("f2", &some_other_function);

// binds just the member function
lua["m1"] = &some_class::member_function;

// binds the class to the type
lua.set_function("m2", &some_class::member_function, some_class{});

// binds just the member variable as a function
lua["v1"] = &some_class::variable;

// binds class with member variable as function
lua.set_function("v2", &some_class::variable, some_class{});
```

lua中使用上面的函数：
```lua
f1() -- some function!
f2() -- some other function!

-- need class instance if you don't bind it with the function
print(m1(sc)) -- 24.5
-- does not need class instance: was bound to lua with one 
print(m2()) -- 24.5

-- need class instance if you 
-- don't bind it with the function
print(v1(sc)) -- 30
-- does not need class instance: 
-- it was bound with one 
print(v2()) -- 30

-- can set, still 
-- requires instance
v1(sc, 212)
-- can set, does not need 
-- class instance: was bound with one 
v2(254)

print(v1(sc)) -- 212
print(v2()) -- 254
```

PS：不能直接在lua中操作用户数据（比如调用本例中的结构体实例`sc.variable`），因此才对需要在lua中调用的成员进行绑定，即`v1`、`v2`就是对成员变量`variable`的`getter/setter`方法；
```console
[sol2] An error occurred and has been passed to an error handler: sol: runtime error: test.lua:4: attempt to index global 'sc' (a userdata value)
```

### 模拟lua成员函数调用

调用lua中的成员函数时，需要将lua对象作为第一个参数传入；

```lua
some_table = { some_val = 100 }

function some_table:add_to_some_val(value)
    self.some_val = self.some_val + value
end

function print_some_val()
    print("some_table.some_val = " .. some_table.some_val)
end
```

```cpp
// do some printing
lua["print_some_val"]();
// 100

sol::table self = lua["some_table"];
self["add_to_some_val"](self, 10);
lua["print_some_val"]();
```

## 返回值

### lua多个返回值

```cpp
lua.script("function f (a, b, c) return a, b, c end");

std::tuple<int, int, int> result;
result = lua["f"](100, 200, 300);
// result == { 100, 200, 300 }
int a;
int b;
std::string c;
sol::tie(a, b, c) = lua["f"](100, 200, "bark");
c_assert(a == 100);
c_assert(b == 200);
c_assert(c == "bark");
```

### cpp多个返回值

```cpp
lua["f"] = [](int a, int b, sol::object c) {
	// sol::object can be anything here: just pass it through
	return std::make_tuple(a, b, c);
};

std::tuple<int, int, int> result = lua["f"](100, 200, 300);
const std::tuple<int, int, int> expected(100, 200, 300);
c_assert(result == expected);

std::tuple<int, int, std::string> result2;
result2 = lua["f"](100, 200, "BARK BARK BARK!");
const std::tuple<int, int, std::string> expected2(100, 200, "BARK BARK BARK!");
c_assert(result2 == expected2);

int a, b;
std::string c;
sol::tie(a, b, c) = lua["f"](100, 200, "bark");
c_assert(a == 100);
c_assert(b == 200);
c_assert(c == "bark");
```

```lua
a, b, c = f(150, 250, "woofbark")
assert(a == 150)
assert(b == 250)
assert(c == "woofbark")
```

## 数据类型

大致可以分为两类：
1. 非用户数据类型：
	- 原始数据类型：`int/float/bool/char`等；
	- 字符串类型：`std::string, const char*`；
	- 函数类型：函数指针，`lua_CFunction, std::function, sol::function, sol::coroutine`，成员变量，成员函数等；
	- 透明数据类型：`sol::variadic_arg, sol::this_state, sol::this_environment`；
	- 用户自定义类型：`sol::usertype, usertype<T>`；
2. 用户数据类型；

### cpp结构体/类

```cpp
struct Doge {
	int tailwag = 50;

	Doge() {
	}

	Doge(int wags)
	: tailwag(wags) {
	}

	~Doge() {
		std::cout << "Dog at " << this << " is being destroyed..." << std::endl;
	}
};
```

```cpp
// fresh one put into Lua
lua["dog"] = Doge{};
// Copy into lua: destroyed by Lua VM during garbage collection
//拷贝出的实例由Lua虚拟机负责GC
lua["dog_copy"] = dog;
// OR: move semantics - will call move constructor if present instead
// Again, owned by Lua
lua["dog_move"] = std::move(dog);
lua["dog_unique_ptr"] = std::make_unique<Doge>(25);
lua["dog_shared_ptr"] = std::make_shared<Doge>(31);

// Identical to above
Doge dog2{ 30 };
lua.set("dog2", Doge{});
lua.set("dog2_copy", dog2);
lua.set("dog2_move", std::move(dog2));
lua.set("dog2_unique_ptr", std::unique_ptr<Doge>(new Doge(25)));
lua.set("dog2_shared_ptr", std::shared_ptr<Doge>(new Doge(31)));

// Note all of them can be retrieved the same way:
Doge& lua_dog = lua["dog"];
Doge& lua_dog_copy = lua["dog_copy"];
Doge& lua_dog_move = lua["dog_move"];
Doge& lua_dog_unique_ptr = lua["dog_unique_ptr"];
Doge& lua_dog_shared_ptr = lua["dog_shared_ptr"];
c_assert(lua_dog.tailwag == 50);
c_assert(lua_dog_copy.tailwag == 30);
c_assert(lua_dog_move.tailwag == 30);
c_assert(lua_dog_unique_ptr.tailwag == 25);
c_assert(lua_dog_shared_ptr.tailwag == 31);
```

结构体/类对象的引用与指针：
```cpp
// The following stores a reference, and does not copy/move
// lifetime is same as dog in C++
// (access after it is destroyed is bad)
lua["dog"] = &dog;
// Same as above: respects std::reference_wrapper
lua["dog"] = std::ref(dog);
// These two are identical to above
lua.set( "dog", &dog );
lua.set( "dog", std::ref( dog ) );


Doge& dog_ref = lua["dog"]; // References Lua memory
Doge* dog_pointer = lua["dog"]; // References Lua memory
Doge dog_copy = lua["dog"]; // Copies, will not affect lua
```

### 映射cpp类到lua

使用`new_usertype<T>`；

```cpp
class MyClass {
public:
    MyClass(int value) : value(value) {}

    void setValue(int newValue) {
        value = newValue;
    }

    int getValue() const {
        return value;
    }

private:
    int value;
};
```

```cpp
// 注册 MyClass 类到 Lua
lua.new_usertype<MyClass>("MyClass",
	"new", sol::constructors<MyClass(int)>(),
	"setValue", &MyClass::setValue,
	"getValue", &MyClass::getValue
);
```

```lua
myObject = MyClass.new(10)
print(myObject:getValue())  -- 输出: 10
myObject:setValue(20)
print(myObject:getValue())  -- 输出: 20
```

## 变量

### 读取

```lua
config = {
	fullscreen = false,
	resolution = { x = 1024, y = 768 }
}
```

支持嵌套读取（但是不安全）：
```cpp
bool isfullscreen = lua["config"]["fullscreen"]; // can get nested variables
sol::table config = lua["config"];
```

可以先用`valid()`来判断该字段是否存在：
```cpp
auto bark = lua["config"]["bark"];
if (bark.valid())
	...
```

也可以使用`std::optional`来检查该字段是否存在，以及该字段的类型：
```cpp
sol::optional<int> not_an_integer = lua["config"]["fullscreen"];
if (not_an_integer) {
	// Branch not taken: value is not an integer
}

sol::optional<bool> is_a_boolean = lua["config"]["fullscreen"];
if (is_a_boolean) {
	// Branch taken: the value is a boolean
}

sol::optional<double> does_not_exist = lua["not_a_variable"];
if (does_not_exist) {
	// Branch not taken: that variable is not present
}
```

如果字段不存在，可以使用`get_or`来指定默认值：
```cpp
int is_defaulted = lua["config"]["fullscreen"].get_or(24);
c_assert(is_defaulted == 24);
```

### 写入



## 执行lua

### 直接执行

```cpp
luaState.script("print('hello world')");

luaState.script_file("test.lua");
```
### 先载入再执行

大部分情况下不需要这样执行lua代码，因为载入需要占用内存；

```cpp
sol::load_result loadRes = luaState.load_file("test.lua");
loadRes();

sol::load_result loadRes = luaState.load_file("test.lua");
loadRes();
```

### 安全性

`script`函数存在**非安全**和**安全**两个版本，如果声明了`#define SOL_ALL_SAFETIES_ON 1`，则会自动调用安全版本，否则调用的是非安全版本；

![250](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240411155445.png)

### 获取错误信息

```cpp
auto res = luaState.script("printf('hello world')", [](lua_State*, sol::protected_function_result pfr) { return pfr; });
if (!res.valid())
{
	sol::error err = res;
	std::cout << err.what() << std::endl;
}
```

```console
[string "printf('hello world')"]:1: attempt to call global 'printf' (a nil value)
stack traceback:
        [string "printf('hello world')"]:1: in main chunk
```

# 高级特性

## 自定义类型

需要覆写`sol_lua_check`, `sol_lua_get`, `sol_lua_push`, `sol_lua_check_get`这几个接口；

```cpp
struct two_things
{
	int a;
	bool b;
}
```

```cpp
template <typename Handler>
bool sol_lua_check(sol::types<two_things>, lua_State* L, int index, Handler&& handler, sol::stack::record& tracking) {
	// indices can be negative to count backwards from the top of the stack,
	// rather than the bottom up
	// to deal with this, we adjust the index to
	// its absolute position using the lua_absindex function
	int absolute_index = lua_absindex(L, index);
	// Check first and second second index for being the proper types
	bool success = sol::stack::check<int>(L, absolute_index, handler) && sol::stack::check<bool>(L, absolute_index + 1, handler);
	tracking.use(2);
	return success;
}

two_things sol_lua_get(sol::types<two_things>, lua_State* L, int index, sol::stack::record& tracking) {
	int absolute_index = lua_absindex(L, index);
	// Get the first element
	int a = sol::stack::get<int>(L, absolute_index);
	// Get the second element,
	// in the +1 position from the first
	bool b = sol::stack::get<bool>(L, absolute_index + 1);
	// we use 2 slots, each of the previous takes 1
	tracking.use(2);
	return two_things { a, b };
}

int sol_lua_push(lua_State* L, const two_things& things) {
	int amount = sol::stack::push(L, things.a);
	// amount will be 1: int pushes 1 item
	amount += sol::stack::push(L, things.b);
	// amount 2 now, since bool pushes a single item
	// Return 2 things
	return amount;
}
```

## metatables

在Lua中，`metatable`是一种特殊的表，它提供了一种机制，允许用户改变表的行为。每个表都可以有一个`metatable`，这个`metatable`中可以包含特殊的键，这些键关联了特定的函数，这些函数会在对原表进行特定操作时被调用，从而改变表的默认行为；

1. **操作重载**：通过metatable，可以重载Lua中的标准操作符（如加法`+`，索引操作`[]`等），使得对用户自定义类型的操作更加自然和直观；

2. **行为修改**：可以修改或扩展表的默认行为，例如，通过`__index`和`__newindex`元方法，可以控制对表中不存在的键的访问和赋值行为；

3. **封装和抽象**：通过隐藏实现细节，可以为C++中的类或对象提供更加“Lua风格”的接口，使得Lua代码更加简洁和易于理解；

4. **自动垃圾回收**：通过`__gc`元方法，可以为C++中的对象提供自动垃圾回收功能，确保资源被适时释放；

