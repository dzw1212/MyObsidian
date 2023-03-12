# 修饰变量

限制该变量为**常量**，初始化之后其值不可再改变；
对应的，`const`修饰的变量必须在初始化时赋初值；

# 修饰指针

## const位于`*`的左侧

表示const修饰的是指针指向的变量，不能通过该指针修改变量的值；
```c++
int nVal = 11;
const int* pVal = &nVal;

*pVal = 12; //Error！

nVal = 13; //本身的值可以修改
std::cout << *pVal << std::endl; //13
```

## const位于`*`的右侧

表示const修饰的是指针本身，该指针不能再指向其他变量；
```c++
int nVal = 11;
int* const pVal = &nVal;

pVal = &nVal; //Error!
```

# 修饰函数

## const位于函数的左侧

表示返回值为常量；

### 返回值为指针

必须使用const的指针来接收函数返回值；
```c++
const int* GetVal()
{
	int nVal = 10;
	return &nVal;
}

int* pVal = GetVal(); //无法从“const int *”转换为“int *”
```

### 返回值为实值

由于函数会把返回值复制到外部临时的存储单元中，加const修饰没有任何价值；
```c++
const int GetVal()
{
	int nVal = 10;
	return nVal;
}

int nVal2 = GetVal(); //OK
```

## const位于函数的右侧

1. 只允许修饰类的非静态成员函数，称为**常函数**；
2. 实际上是对`this`指针的修饰，限制该函数内不可修改成员的值（除了`mutable`成员）；
3. 类的常对象只能调用常函数；

