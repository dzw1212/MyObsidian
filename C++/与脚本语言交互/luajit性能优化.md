
PS：luajit分为**jit**模式和**interpreter**模式，在`ios`下会变为interpreter模式，性能接近原生lua，以下各个优化方案大多不适合，不需要过分优化；
# 减少不可预测的分支代码

luajit使用了`trace compiler`的特性 —— 不是直接将lua代码编译成机器码，而是先编译成字节码并运行，针对热点代码做性能分析，如果发现某段代码经常运行，就会针对该段代码进行性能优化，并默认优化后的代码为执行代码；

如果经常出现不可预测的代码，就会`miss`luajit的优化代码，跳转出去，再根据情况看要不要再编译一段新的机器码来适配新的情况；

因此建议使用`math.min/max`或者`bitop`等方法来避免`if/else`的写法；

以下有一些避免`if/else`的方法：
## 使用Table进行函数映射

```lua
function doSomethingA()
    print("Doing something A")
end

function doSomethingB()
    print("Doing something B")
end

local actions = {
    ["conditionA"] = doSomethingA,
    ["conditionB"] = doSomethingB,
}

local condition = "conditionA"
actions[condition]()
```

## 更多的使用逻辑运算符

```lua
local condition = true
local result = condition and "True Result" or "False Result"
print(result)
```

# 使用FFI实现数据结构而非Table

比如分别用`FFI`和`Table`实现一个`Vertex3`结构体：
```lua
--FFI实现
local ffi = require("ffi")

ffi.cdef[[
typedef struct {
    double x, y, z;
} Vertex3;
]]

-- 创建一个Vertex3实例
local vertex = ffi.new("Vertex3", {1.0, 2.0, 3.0})
```

```lua
--Table实现
function Vertex3(x, y, z)
    return {x = x, y = y, z = z}
end

-- 创建一个Vertex3实例
local vertex = Vertex3(1.0, 2.0, 3.0)
```

lua的Table本质是一个`hash table`，在`hash table`访问字段固然是缓慢的并且要存储大量额外的东西。而`FFI`可以做到只分配xyz三个float的空间就能表示一个`Vector3`，自然内存占用要低得多，而且luajit会利用`FFI`的信息，实现访问xyz的时候直接读内存，而不是像`hash table`那样走一次`key hash`，性能也高得多；

# 尽可能通过FFI调用c函数

使用`FFI`导出c函数，你需要提供c函数的原型，有了c函数的原型信息，luajit可以知道每个参数的准确类型、返回值的准确类型；

```cpp
extern "C" {
    void printMessage(const char* message) {
        std::cout << message << std::endl;
    }
}
```

编译为动态库：
```console
g++ -shared -o libexample.so example.cpp -fPIC
```

```cpp
local ffi = require("ffi")

ffi.cdef[[
void printMessage(const char* message);
]]

local lib = ffi.load("./libexample.so")
lib.printMessage("Hello from LuaJIT!")
```

# 避免使用pairs

- **`pairs`** 遍历表中的所有元素，包括索引为非整数的元素。它不保证遍历顺序；

- **`ipairs`** 仅遍历表中索引为连续整数的部分（从1开始），直到第一个`nil`值为止。它按照索引的升序遍历这些元素；

luajit暂时不支持`for k, v in pairs(tab)`，这种遍历方式难以生成机器码，直接使用数值做索引是更推荐的做法；

# 考虑手动展开循环

比较现代的编译器都拥有了在运行时展开循环次数较少的循环的能力，但是如果大循环中嵌套了小循环，编译器会误判循环次数，从而选择不展开循环；

因此，可以考虑在编写代码时手动展开循环次数较少的循环（尤其当其位于其他循环内的时候）；

# 更推荐local函数而非global

```lua
local math_sin = math.sin

local function test()
	math.sin(1)
	math_sin(1)
end
```

上面的代码示例中`math_sin`相当于对一个`global`函数进行了一次`local`缓存，而调用`math.sin`需要在全局空间中搜索`math`，因此`math_sin`效率更佳；

- 如果一个函数仅在本文件中使用，将其声明为`local`；
- 如果是一个全局函数，则在文件开头处`local`缓存一下；

# 减少local变量的数量

如果一个变量仅使用一两次就不再使用，尽量不要声明一个`local`变量去容纳它；

比如：
```lua
local c = a + b
z = x[c] + y[c]

--prefer
z = x[a+b] + y[a+b]
```

因为luajit的JIT编译器设计时有一些限制，其中之一就是对函数中local变量的数量有限制。这个限制主要是出于性能和资源管理的考虑。当local变量的数量超过一定阈值（例如在某些版本中是200个）时，JIT编译器可能就无法处理这样的函数；

# 一些尚未实现的功能

luajit的文档中有一些`NYI(Not Yet Implement)`的功能，这些功能尚未JIT化，因此效率较低；

截至2023年，LuaJIT的NYI特性包括但不限于以下几点：

1. **复杂的数据类型作为table的键**：使用复杂数据类型（如函数、线程、用户数据）作为table的键时，可能会导致JIT编译失败。

2. **某些table操作**：比如`table.insert`和`table.remove`在特定情况下可能不会被JIT编译。

3. **某些math库函数**：如`math.log`和`math.atan2`等在某些参数下可能不会被JIT编译。

4. **协同程序（coroutines）**：LuaJIT对协同程序的支持有限，特别是在协同程序之间进行复杂交互时，可能会导致JIT编译失败。

5. **动态类型转换**：在某些情况下，频繁的动态类型转换可能会导致JIT编译失败。

6. **复杂的控制流**：如过于复杂的循环、递归调用和长函数等，可能会导致JIT编译失败。

7. **调试和元表操作**：使用`debug`库的某些功能，以及动态修改元表（`metatable`）可能会导致JIT编译失败。

8. **FFI回调**：使用外部函数接口（`FFI`）库时，回调到Lua的函数可能不会被JIT编译。
