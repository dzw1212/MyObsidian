# 01 . 将C++看作一个语言联邦

C++是一个由相关语言组成的联邦而非单一语言；

- C语言：C++以C为基础，区块`Block`、语句`Statement`、预处理器`Preprocessor`、内置数据类型`Build-in Data Type`、数组`Array`、指针`Pointer`等概念统统来源于C，但是C没有模板`Template`、没有异常`Exception`、没有重载`Overloading`等；

- Oject-Oriented C++：即类`Class`、封装`Encapsulation`、继承`Inheritance`、多态`Polymorphism`、虚函数等；

- Template C++：由于模板威力强大，其带来了新的编程范型，也就是所谓的模板元编程（`Template Meta Programming`，`TMP`）；

- STL：STL有自己特殊的办事方式，使用时必须要遵守其规则；

比如，对于内置类型，通常`pass-by-value`比`pass-by-ref`更高效，但对于用户自定义类型，`pass-by-ref-const`往往是更好的选择；但在STL中，迭代器和函数对象都是在C指针之上塑造出来的，所以对STL的迭代器和函数对象而言，`pass-by-value`又成了更好的选择；

因此C++并不是带有一组守则的一体语言，其每个次语言都有自己的规约；

# 02 . 尽量用const,enum,inline代替#define

`#define`的宏定义名称也许在编译器开始处理源代码之前就被预处理器移走了，可能不会被记入符号表（`Symbol Table`），不利于debug；

## class专属常量

为了将常量的作用域限制在类中，必须使它成为类的成员，并且为了确保其只有一个实体，需要用`static`限制：

```cpp
class GamePlayer
{
public:
	int GetNumTurns() { return NumTurns; }

private:
	static const int NumTurns = 5; //虽然定义了初值，但仍属于声明式，不是定义式
	int scores[NumTurns];
};

const int GamePlayer::NumTurns; //定义式，声明时已赋值 
```

如果只读取值而不需要取地址，则允许只有声明式而没有定义式；

`#define`并不重视作用域，一旦宏被定义，除非被`#undef`，否则其就在之后的编译过程中有效；

## the enum hack

一些旧的编译器不支持静态成员变量在声明时赋值，必须在定义中才能赋值，这导致了一个问题：如果类的成员在编译期间需要使用该变量的值（比如上面代码中的`int score[NumTurns]`），就无法编译通过；

```cpp
class GamePlayer
{
public:
	int GetNumTurns() { return NumTurns; }

private:
	static const int NumTurns;
	int scores[NumTurns]; //error C2131: 表达式的计算结果不是常数
};

const int GamePlayer::NumTurns = 5;
```

此时可以使用一种名为`the enum hack`的做法，其依据是：一个枚举类型的数值可以被当作整型使用；

```cpp
class GamePlayer
{
private:
	enum{ NumTurns = 5 };
	int scores[NumTurns];
};
```

`the enum hack`的优点：
1. `the enum hack`的行为与`#define`非常相似，在绝对不会导致非必要的内存分配的同时，实现了作用域的限制，如果你不想让被人获得一个指针或引用指向该变量，使用`the enum hack`非常合适；
2. 许多代码使用了这种技术，事实上它属于模板元编程的基础技术；

## 使用Template Inline代替#define函数

`#define`还有一种用法是用来实现宏`macros`，从而避免函数调用的额外开销；

```cpp
#define CALL_WITH_MAX(a, b) testFunc(((a) > (b)) ? (a) : (b))
```

这种方式有诸多缺点：
1. 需要为每个参数都加上小括号，否则会产生运算优先级问题，非常得麻烦；
2. 即使加上了小括号，也无法避免一些问题（比如多次重复调用）；

比如：
```cpp
int a = 5;
CALL_WITH_MAX(++a, 0) //这种情况下a累加了2次
CALL_WITH_MAX(++a, 5) //这种情况下a累加了1次
```

更合理的方法是使用`Template Inline`函数：

```cpp
template<typename T>
inline void callWithMax(const T& a, const T& b)
{
	testFunc(a > b ? a : b);
}
```

# 03 . 尽可能使用const

如果`const`出现在星号左边，表示限制的是常量；
如果`const`出现在星号右边，表示限制的是指针自身；

令函数返回常量值，往往可以降低因客户端错误而造成的意外；
比如以下的这个场景：

```cpp
class Rational
{
public:
	float fVal;

public:
	Rational& operator=(float f) 
	{
		this->fVal = f;
		return *this;
	}

	Rational& operator*(const Rational& other)
	{
		this->fVal *= other.fVal;
		return *this;
	}
};

int main() 
{
	Rational a, b, c;
	a = 1.f;
	b = 2.f;
	c = 3.f;

	(a * b) = c; //客户端的异常调用

	//结果是a=3,b=2,c=3，具体的结果取决于operator*和operator=中的实现 
    return 0;
}
```

这种情况下可以用`const`对`operator*`进行限制，以避免异常操作导致的错误；

```cpp
const Rational& operator*(const Rational& other)
{
	this->fVal *= other.fVal;
	return *this;
}
```

![400](https://pic-1315225359.cos.ap-shanghai.myqcloud.com/20240213175404.png)

## 推荐logical const而不是bitwise const

`bitwise const`指的是，成员函数只有在不更改对象的任何成员变量（`static`变量除外，其存储在全局数据区，而非每个对象的内存中）时才可以说是`const`，即不更改对象内的任何一个`bit`；

然而，有一些成员函数虽然不是完全的`const`，却能通过`bitwise`测试 —— 一个修改了“指针所指物”的成员函数，不符合`const`的定义，但由于只有“指针”属于成员，因此不会违反`bitwise const`；

```cpp
const CTextBlock cctb("hello");
char* pc = &cctb[0];

*pc = 'A'; //改变了指针指向的值，但没有报错
```

与之对应的是，`logical const`允许函数内部改变了一些成员变量的值，只要这些改变不影响对象对外的观察结果，该对象仍然被认为是常量。编译器一般不接受`logical const`，因此通常通过将一些成员变量声明为`mutable`来实现，这样即使在`const`成员函数中也可以修改它们；

[[为什么需要mutable]]

## 在non-const版本中调用const版本

一种非常常见的情况，就是一个函数同时需要`const`和`non-const`的版本，为了避免代码的重复与维护的困难，需要一种在`non-const`版本函数中调用`const`版本函数的方法、能够将常量性移除的方法，即`const_cast`；

为什么是`non-const`中调用`const`，而不是`const`中调用`non-const`？
	因为`const`成员函数可以在更多的上下文中被调用，包括`const`对象和`non-const`对象上，而`non-const`成员函数只能在`non-const`对象上被调用；

```cpp
class TextBlock
{
public:
	char* text;

	const char& operator[](std::size_t pos) const //const版本
	{
		return text[pos];
	}

	char& operator[](std::size_t pos) //non-const版本
	{
		//先将this指针转为const，再将const版本的operator[]的结果去除const
		return const_cast<char&>(static_cast<const TextBlock&>(*this)[pos]);
	}
};
```

# 04 . 确定对象被使用前已被初始化

C++并不保证一定初始化（比如C风格的`array`中的值往往不会初始化，而`vector`中的往往保证会初始化），因此最佳的方法就是：永远在对象被使用前将其手动初始化；

## 成员初始化列表的优点

相较于在构造函数中使用赋值操作`=`来初始化成员变量，更推荐使用成员初始化列表`member initializer list`；

1. **效率更高**：成员初始化列表直接初始化成员变量，而不是先调用默认构造函数然后再赋值。对于非内置类型的成员变量，这意味着避免了不必要的构造和析构过程，从而提高了效率；

2. **允许初始化常量与引用成员**：常量成员变量和引用类型成员变量必须在成员初始化列表中初始化，因为它们一旦被默认构造之后就不能再被赋值；

3. **允许初始化没有默认构造函数的类类型成员**：如果一个类成员没有默认构造函数，你必须在成员初始化列表中对它进行初始化；

4. **初始化顺序明确**：成员变量的初始化顺序与它们在类定义中的声明顺序一致，而不是它们在成员初始化列表中出现的顺序。这使得初始化过程更加清晰和可预测；

5. **可以调用基类的构造函数**：通过成员初始化列表，可以明确地选择调用基类的哪个构造函数，这对于基类有多个构造函数的情况特别有用；

```cpp
//不推荐，需要先构造default construct再赋值assign
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones)
{
	theName = name;
	theAddress = address;
	thePhones = phones;
	numTimes = 0;
}

//使用成员初始化列表，仅一次copy构造
ABEntry::ABEntry(const std::string& name, const std::string& address, const std::list<PhoneNumber>& phones) : theName(name), theAddress(address), thePhones(phones), numTimes(0) {}

//如果就是想要default construct，不使用参数初始化即可
theName(), theAddress(), thePhones()
```

## 初始化顺序

- 基类总是先于派生类被初始化；
- 成员变量的初始化顺序取决于它们的声明顺序；

==为了避免歧义，建议初始化列表中的顺序严格按照成员变量的声明顺序==；

## 不同编译单元的non-local static对象的初始化顺序

- **编译单元**`translation unit`：指产出单一目标文件`Single Object File`的源码，即单一代码文件加上其所包含的头文件；
- **`non-local static`对象**：指的是那些不在函数内部、类定义内部、匿名命名空间内部声明的静态对象，包括：
	- 1. **全局静态对象**：在所有函数外部定义的静态对象；
	- 2. **静态类成员变量**：在类定义内部声明为`static`的成员变量，其生命周期从程序开始直到程序结束；
	- 3. **在命名空间内声明的静态对象**：不属于任何函数和类；

`non-local static`对象的初始化顺序在不同的编译单元之间是未定义的，这意味着，如果一个`non-local static`对象依赖另一个`non-local static`对象的顺序，则会导致所谓的==静态初始化顺序问题==；

解决方法：**将每个`non-local static`对象放在其专属的接口函数内**（类似单例模式）；
换言之，即是将`non-local static`对象转为了`local static`对象，而`local static`对象只会在其被第一次调用时初始化；更棒的是，如果始终不使用该接口，就不会触发构造和析构函数，避免了额外的开销；

这种接口被称为`reference-returning`函数：

```cpp
//头文件中，比如Utility.h
MyClass& getMyClassInstance();

//Utility.cpp
#include "MyClass.h"
#include "Utility.h"

MyClass& getMyClassInstance()
{
	static MyClass instance;
	return instance;
}


//其他需要使用该接口的文件
#include "Utility.h"

int main()
{
	auto ins = getMyClassInstance();
	return 0;
}
```

但是这种形式的安全性只在单线程下成立，如果涉及到多线程，无论是`local`还是`non-local`的静态对象，在多线程环境下等待都会带来麻烦，因此多线程下更安全的做法是 —— **在程序的单线程启动阶段手动调用所有的`reference-returning`函数**；


# 05 . 了解C++默认提供并调用了哪些函数

一般来说，如果你声明了一个空的类且什么函数都不声明，则C++会自动为你补充：
	1. `default`构造函数
	2. `copy`构造函数
	3. `copy assign`操作符
	4. 析构函数
以上这些函数都是`public`且`inline`的；

```cpp
class Empty
{
public:
	Empty() {...} //default构造函数
	~Empty() {...} //析构函数
	Empty(const Empty& rhs) {...} //copy构造函数
	Empty& operator=(const Empty& rhs) {...} //copy assign操作符
}
```

这些函数只有被调用时，才会被编译器创建出来：

```cpp
Empty e1; //调用default构造函数
Empty e2(e1); //调用copy构造函数
e2 = e1; //调用copy assign操作符
```

有以下几种情况，编译器不会生成`copy assign`操作符：
1. 类中包含`const`或引用成员；
2. 基类的拷贝复制操作符为`private`或`protected`；
3. 类中有成员变量的类型没有默认的拷贝构造操作符；

# 06 . 如果不想让编译器生成函数应显式拒绝

比如单例模式中，希望类的示例是独一无二的、不能拷贝的，你可以将`copy`构造函数与`copy assign`操作符设为`private`，但是这并不绝对安全，因为其`member`函数和`friend`函数还是可以调用到，为此，有种做法是**将这些函数的声明放在`private`中且故意不实现它们**（比如`ios_base`，`basic_ios`和`sentry`等标准程序库），这样的话如果`member`或`friend`函数中尝试拷贝，则链接器会报错；

可以实现一个这样的基类，其派生类都会禁用拷贝（类似`boost`库中的`noncopyable`类）：

```cpp
class Uncopyable
{
protected:
	//如果放在public中，则外部代码会直接创建Uncopyable的实例
	//如果放在private中，则其派生类也无法创建和销毁
	Uncopyable() {} 
	~Uncopyable() {}
private:
	Uncopyable(const Uncopyable&); //只声明，不实现
	Uncopyable& operator=(const Uncopyable&); //只声明，不实现
}
```

在C++11之后，可以使用`= delete`显式禁用；

```cpp
class MyClass {
private:
    MyClass(const MyClass&); // 私有拷贝构造函数
    MyClass& operator=(const MyClass&); // 私有拷贝赋值运算符
public:
    MyClass() {} // 默认构造函数
};
```

# 07 . 为多态基类声明virtual析构函数

如果基类的析构函数不是`virtual`，那么当通过基类指针删除派生类对象时，只会调用基类的析构函数，而不会调用派生类的析构函数。这在基类和派生类都管理资源时尤其危险，因为派生类的资源可能不会被释放；

一般来说基类中会有很多的虚函数，这些==虚函数的目的是实现派生类的标准定制化==；

在添加虚函数后，类中会自动生成虚函数表`Virtual Table`和虚表指针`Virtual Table Pointer`，因此使用虚函数会导致类所占用的内存大小增加，假如是`Point2D`或`Veterx3`这样的需要和其他系统交互的类，应保持固定的、公认的大小，不应该使用虚函数；

因此，只有当类中至少有一个虚函数，才能没有负担地将析构函数设为`virtual`；

综上所述，**在需要通过基类指针操纵派生类对象时，不应该继承自一个含有`non-virtual`析构函数的类**；

假如基类的析构函数不仅是一个虚函数，还是一个纯虚函数，那么==即使纯虚函数不需要提供定义，也应该为基类创建一个析构函数的定义==，因为析构时会派生类逐层往下到基类调用析构函数，如果基类没有析构函数的定义则链接器会报错；

# 08 . 别让异常逃离析构函数

1. **可能导致资源泄漏**：如果析构函数抛出异常，而它是在处理另一个异常时被调用的，那么程序可能会直接终止（C++标准规定处理一个异常时抛出另一个异常会调用`std::terminate`），这可能会导致资源泄漏，因为之后的资源没有被释放；
2. 析构函数的主要任务是释放资源和清理，这些操作应该尽可能不失败，如果存在想象之内的异常发生情况，应该设计别的方式来处理异常；

一般推荐的做法是：

```cpp
class DBConnect
{
public:
	void close()
	{
		db.close();
		closed = true;
	}
	~DBConnect()
	{
		if (!closed)
		{
			try
			{
				db.close();
			}
			catch (...)
			{
				log...; //记录异常
				std::abort(); //或者直接终止
			}
		}
	}
}
```

提供给客户处理错误的机会，在析构函数中进行二次判断，同时记录异常，或者主动终止程序（总比不知在什么时候被动终止要好）；


# 09 . 绝不在构造和析构过程中调用virtual函数

比如以下的这个例子：

```cpp
class Transaction
{
public:
	virtual void logTransaction() const;
	Transaction()
	{
		...
		logTransaction(); //构造函数中调用虚函数
	}
}

class BuyTransaction : public Transaction
{
public:
	virtual void logTransaction() const;
}

class SellTransaction : public Transaction
{
public:
	virtual void logTransaction() const;
}
```

此时创建一个`BuyTransaction`，由于基类先于派生类构造的缘故，调用的`logTransaction`会是基类中的，而不是派生类的；

```cpp
BuyTransaction buy;
```

# 10 . 令operator=返回ref to \*this

为了==实现连锁赋值==，赋值操作符需要返回一个指向操作符左侧实参的引用；

```cpp
int x, y, z;
x = y = z = 15;

//上面的连锁赋值，根据右侧结合律，被解析为
x = (y = (z = 15));
```

这也适用于`+=`、`-=`、`*=`、`/=`等一系列与赋值相关的操作符；

```cpp
class Widget
{
public:
	Widget& operator=(int rhs)
	{
		...
		return *this;
	}
	
	Widget& operator+=(const Widget& rhs)
	{
		...
		return *this;
	}
}
```


# 11 . 在operator=中处理自我赋值

一般来说自我赋值是没有必要的，但是有时候很难完全避免自我赋值，比如`a[i] = a[j]`，或者基类指针与派生类指针同时指向一个派生类实例；在某些情况下，自我赋值并不安全，比如：

```cpp
class Bitmap;

class Widget
{
private:
	Bitmap* pb;
public:
	Widget& operator=(const Widget& rhs)
	{
		delete pb;
		pb = new Bitmap(*rhs.pb); //此时rhs.pb已被delete
		return *this;
	}
}
```

为了避免这种情况，通常会在赋值操作符中加一个自我检测：

```cpp
Widget& operator=(const Widget& rhs)
{
	if (this == &rhs) //自我检测
		return *this;

	delete pb;
	pb = new Bitmap(*rhs.pb); //此时rhs.pb已被delete
	return *this;
}
```

但这样做并不是好的方式，首先自我赋值是少数情况，不应该为了少数情况拖慢整体的效率；其次如果在`new Bitmap`的过程中出现异常，则指针`pb`指向的是一块已被删除的不安全的区域；真正好的做法是，不做自我检测，而是通过设计代码写法避免这种问题：

```cpp
Widget& operator=(const Widget& rhs)
{
	Bitmap* pCopy = pb;
	pb = new Bitmap(*rhs.pb); //先创建，没问题了再删除原值
	delete pCopy;
	return *this;
}
```

## copy and swap技术

另外更好的方式是一种叫做`copy and swap`的技术，这是实现赋值操作符的一种常用技术，它通过使用拷贝构造函数和析构函数来简化代码，在处理自我赋值的同时确保异常安全 —— 创建一个临时对象，交换自身与临时对象的数据，然后销毁临时数据；

```cpp
class Widget
{
public:
	void swap(Widget& ths);

	Widget& operator=(const Widget& rhs)
	{
		Widget temp(rhs);
		swap(temp);
		return *this;
	}
}
```

# 12 . 复制对象时不要忘记任何一个成员

当用户自己写复制函数（copy构造函数+copy assign操作符）时，如果遗漏了成员编译器并不会有提醒，因此当自己编写复制函数时：
1. 复制所有的local成员变量；
2. 调用所有基类的适当的复制函数；

```cpp
class PriorityCustomer : public Customer
{
private:
	int priority;
public:
	PriorityCustomer(const PriorityCustomer& rhs)
	: Customer(rhs), priority(rhs.priority) {}

	PriorityCustomer& operator=(const PriorityCustomer& rhs)
	{
		Customer::operator=(rhs);
		priority = rhs.priority;
		return *this;
	}
}
```

通常`copy`构造函数和`copy assign`操作符会有大量的代码重复，可以使用`copy and swap`技术来消除代码重复；

# 13 . 使用对象来管理资源

依靠逻辑代码种主动`delete`来释放资源是不可靠的，这个释放资源的操作可能会因为各种原因不会执行到，为了确保资源能被正确释放，需要将释放资源的操作放进类的析构函数中，在销毁对象的同时自动释放资源；

智能指针`auto_ptr`正是为了这个场景而生的，其在析构函数中会`delete`所指向的对象；

```cpp
//Not recommand
Investment* pInv = createInvestment();
delete pInv;
pInv = nullptr;

//Recommand
std::auto_ptr<Investment> pInv(createInvestment()); //析构时自动删除pInv
```

由于这个机制，一定注意别让多个`auto_ptr`指针指向同一个对象，否则会多次删除同一对象，为了避免这种情况的发生，`auto_ptr`具有这样一个性质 —— 若通过`copy` 构造函数或者`copy assign`操作符复制自身，自身会变成`null`，复制出的指针拥有资源的唯一拥有权；

[[智能指针]]

这种“唯一所有权”的性质有时不适合管理动态分配资源，替代方案是引用计数型智能指针`shared_ptr`，它会持续追踪共有多少对象指向该资源，并在没有指针指向该资源时自动删除该资源；

`shared_pre`的行为类似垃圾回收gc，但是`shared_pre`无法打破环状引用；

~~需要注意的是，无论`auto_ptr`还是`shared_ptr`，其析构时执行的都是`delete`而非`delete []`，因此使用这两个智能指针管理数组资源不是一个好选择~~；

在 C++11 及以后的版本中，`std::shared_ptr `可以安全地用于管理数组资源，但需要在创建 `std::shared_ptr` 时提供自定义的删除器。C++17 进一步增强了对数组的支持，引入了可以直接管理动态数组的`std::shared_ptr`重载，使得使用自定义删除器变得不再必要；

```cpp
// C++11 和 C++14，使用自定义删除器
std::shared_ptr<int> ptr(new int[10], std::default_delete<int[]>());

// C++17 及以后，直接支持数组
std::shared_ptr<int[]> ptr = std::make_shared<int[]>(10);

// C++11后就支持数组
std::unique_ptr<int[]> array(new int[10]);
```

# 14 . 资源管理类中要小心复制行为

假如有一个`Lock`类，其管理一个互斥器对象并遵守`RAII`设计逻辑，在构造函数中获取资源、析构函数中释放资源：

```cpp
class Lock
{
public:
	explicit Lock(Mutex* pm)
	: mutexPtr(pm)
	{
		lock(mutexPtr);
	}

	~Lock()
	{
		unlock(mutexPtr);
	}
private:
	Mutex* mutexPtr;
}
```

有一个潜在的问题是，如果`Lock`被复制了应该怎么处理？

1. 禁用复制：比如将复制函数设为`private`；
2. 使用引用计数管理复制：`shared_ptr`的默认行为是在计数为0时删除所指的对象，幸运的是`shared_ptr`允许指定删除器`deleter`，在计数为0时执行所谓的删除器；

```cpp
explicit Lock(Mutex* pm)
: mutexPtr(pm, unlock) //删除器unlock
{
	lock(mutexPtr.get());
}

private:
	std::shared_ptr<Mutex> mutexPtr;
```

# 15 . 在资源管理类中提供对原始资源的访问

比如通过`shared_ptr`拿到`raw pointer`；

- 显式转换：`shared_ptr`提供了`get()`方法获取原始指针；
- 隐式转换：`shared_ptr`等智能指针重载了指针取值操作符`operator->`和`operator*`，它们都允许隐式转换至原始指针；

显式转换比较安全清晰，隐式转换比较方便，需要按需斟酌；

[[隐式转换]]

# 16 . 成对使用new和delete时要采取相同形式

`new`一个对象对应`delete`；
`new`一个对象数组对应`delete []`

# 17 . 以独立语句将原始指针置入智能指针

假如使用以下形式将原始指针置入智能指针，则可能会出现资源泄漏：

```cpp
int priority();
void processWidget(std::shared_ptr<Widget> pw, int priority);

int main()
{
	processWidget(std::shared_ptr<Widget>(new Widget), priority());
	return 0;
}
```

编译器会执行三个操作：
1. `new Widget`；
2. 生成`shared_ptr`；
3. 调用`priority()`；

这三个操作的执行顺序并不确定，假如调用priority()在生成原始指针、生成智能指针之间，且产生了异常的话，生成的原始指针会遗失，从而产生了资源泄漏；

因此，最好把**将原始指针置入智能指针**这种操作作为独立的一个语句执行；

# 18 . 让接口更容易被正确使用，不易被误用

假如有一个日期类，其接受月、日、年三个参数，如果三个参数的类型都是整形，则用户可能会因为误看而填错，同时编译器不会给出任何提示，这就是一个设计的容易误用的接口；

```cpp
class Date
{
public:
	Date(int mouth, int day, int year);
}

//用户误用
Date d(30, 3, 1995);
Date d(2, 30, 1995);
```

这种情况下可以考虑将月、日、年进行封装，以确保参数足够清晰；

```cpp
struct Day
{
	explicit Day(int d) : val(d) {}
	int val;
}
struct Mouth
{
	explicit Mouth(int d) : val(d) {}
	int val;
}
struct Year
{
	explicit Year(int d) : val(d) {}
	int val;
}

//限制后的调用方式
Data d(Mouth(3), Day(30), Year(1995));
```


# 19 . 设计class如同设计type

# 20 . 使用pass-by-ref-to-const代替pass-by-value

使用`pass-by-ref-to-const`的方式传递参数要高效的多；

只有内置类型和STL的迭代器和函数对象使用`pass-by-value`时效率高；

# 21 . 必须返回对象时，不要试图返回引用

任何时候看到一个引用，都应该立即问自己它的另一个名称是什么，因为**引用不是实体而是别名**；

返回局部变量的引用是愚蠢的（指针同理），因为变量的实体在函数结束后就会被销毁；

```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
	Rational res(lhs.n * rhs.n, lhs.d * rhs.d); //局部变量
	return res;
}
```

试着在`heap`中创建变量并返回引用，但问题是：谁来负责`delete`这个变量？这几乎一定会导致内存泄漏；

```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
	Rational* pRes = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
	return *pRes;
}
```

或许还有一种返回`static`对象的方法，但这种方法在相等判断等处会发生bug；

```cpp
const Rational& operator*(const Rational& lhs, const Rational& rhs)
{
	static Rational res;
	res = lhs.n * rhs.n, lhs.d * rhs.d;
	return res;
}

Rational a,b,c,d;
if ((a * b) == (c * d)) //由于operator*返回的是同一个static变量，因此总是相等的
	...
```

因此，假如一个函数必须返回对象，那就直接返回一个新的对象好了；

# 22 . 将成员变量声明为static

# 23 . 使用非成员函数代替成员函数

# 24 . 若所有参数都需要类型转换，请使用非成员函数

# 25 . 考虑写出一个不抛出异常的swap函数

STL库提供的`swap`函数（前提是类型`T`有`copy`构造函数和`copy assign`操作符）：

```cpp
namespace std
{
	template<typename T>
	void swap(T& a, T& b)
	{
		T temp(a);
		a = b;
		b = temp;
	}
}
```

但在一些特殊情况下，使用`std::swap`不是效率最高的方式，比如：

```cpp
class WidgetImpl
{
private:
	int a, b, c;
	std::vector<double> v;
	... //大量的数据
}


class Widget
{
public:
	Widget(const Widget& rhs);
	Widget& operator=(const Widget& rhs)
	{
		*pImpl = *(rhs.pImpl); //交换两个Widget对象，只需要交换pImpl所指的值即可
		...
	}
private:
	WidgetImpl* pImpl;
}
```

这种情况下，使用标准的`std::swap`反而效率低下；
为了应付这种情况，可以自定义类的高效`swap`函数 + `std::swap`特化以实现高效交换；

```cpp
#include <algorithm> // For std::swap, unless you implement your own version

class Widget {
public:
    // 构造函数、析构函数和拷贝构造函数保持不变

    Widget(const Widget& rhs);
    Widget& operator=(const Widget& rhs)
    {
        swap(rhs); // 交换*this和rhs
        return *this;
    }

    // 实现高效的swap函数
    void swap(Widget& other) noexcept {
	    using std::swap;
        swap(pImpl, other.pImpl); // 仅交换指针，而非指针所指向的大量数据
    }

private:
    WidgetImpl* pImpl;
};

// 特化，使得std::swap可以识别到Widget的swap
namespace std {
    template<>
    void swap<Widget>(Widget& a, Widget& b) noexcept {
        a.swap(b);
    }
}
```

为什么使用`using std::swap`，而不是直接使用`std::swap()`？

这是一种常见的技巧，它允许代码利用==参数依赖查找==（`Argument-Dependent Lookup, ADL`）来选择最合适的`swap`函数。这种方法的目的是确保如果`pImpl`的类型有一个特定的`swap`实现，那么这个特定的实现将被调用，而不是总是调用`std::swap`；

# 26. 尽可能延后变量定义式的出现时间

1. 尽可能在需要使用之前再定义变量，如果函数一开始就定义变量，可能会在中途抛出异常或`return`，导致没有使用变量却付出了构造和析构成本；

2. 绝大部分时候定义一个变量后紧跟着就是赋值操作，最好使用一个初值来初始化变量，因为`copy`构造的效率高于`default`构造 + `copy assign`操作符；

如果变量只在循环中使用，那么把变量定义在循环外每次循环时赋值好，还是应该把变量定义在循环内？
	循环外：1个构造函数 + 1个析构函数 + n个赋值操作；
	循环内：n个构造函数 + n个析构函数；
具体应该选择哪种方式，取决于析构、赋值的效率对比，和循环次数n的大小；


# 27 . 尽量少做转型操作

旧式转型：
	C风格：`(T)expression`
	函数风格：`T(expression)`

新式转型：
	`const_cast`
	`dynamic_cast`
	`reinterpret_cast`
	`static_cast`

[[转型cast]]

---

在许多应用框架中，都要求派生类在执行虚函数时，首先调用其基类的对应虚函数，试看下面这种实现方式：

```cpp
class Window
{
public:
	virtual void OnResize();
}

class SpecialWindow : public Window
{
public:
	virtual void OnResize() override
	{
		static_cast<Window>(*this).OnResize(); //类型转换为基类，再调用虚函数
		...
	}
}
```

这里使用的`static_cast<Window>(*this)`实际上会创建`Window`类的一个临时副本，而不是简单地调用基类中的`OnResize`方法。因此，这段代码不会按预期工作，因为它不会在当前对象的上下文中调用基类的`OnResize`方法，而是在一个临时对象上调用，这个临时对象在调用结束后就被销毁了。

正确的写法应该是：
```cpp
virtual void OnResize() override
{
	Window::OnResize(); // 直接调用基类的OnResize方法
	// 其他操作...
}
```

---

`dynamic_cast`的速度是非常慢的，一般用到`dynamic_cast`的场合都是：手头只有一个基类指针，但是希望在某个特定的派生类身上执行操作：

通常不推荐使用`dynamic_cast`，可以为基类及每个派生类都添加虚函数实现（哪怕是空函数体），这样即使在非目标派生类上执行了操作也不会出错；

绝对需要避免`cascading dynamic_cast`，这种做法会严重拖慢性能，应当修改类的设计以避免这种情形：

```cpp
if (SpecialWindow1* psw1 = dynamic_cast<SpecialWindow1>(pWindow))
{...}
else if (SpecialWindow2* psw2 = dynamic_cast<SpecialWindow2>(pWindow))
{...}
else if ...
```

# 28 . 避免返回handles指向对象内部成分

引用、指针和迭代器都是所谓的“号码牌”（`handles`），返回一个代表内部数据的`handles`会带来降低对象封装性的风险；除此之外，如果用户将`handles`存储起来，而对象不知在什么时候被销毁了，之前存储的`handles`就会指向未定义的数据；

如果确实需要返回指向内部数据的`handles`，请加上`const`加以限制返回值；

# 29 . 为异常安全而努力是值得的

# 30 . 透彻了解inline的里里外外

`inline`函数背后的观念是：对此函数的每一个调用都使用函数本体替换；这么做可能会增加目标码的大小，在一台内存有限的机器上，过于热衷`inline`会造成程序体积过大，从而降低缓存命中率，带来效率的损失；

`inline`只是对编译器的一个申请，而不是强制命令；这个申请可以隐喻提出，也可以明确提出；
	隐喻提出：将函数定义放在类的定义中；
	明确提出：添加`inline`关键词；

大部分编译器拒绝将太过复杂的函数`inlining`（比如带循环和递归的），而所有对虚函数的`inline`请求都会被拒绝，因为虚函数意味着“等待，直到运行期才确定调用哪个函数”，而`inline`意味着“执行前，将调用动作替换为被调用函数的本体”，这两种想法是冲突的；

另一个需要注意的点是，如果f是程序库内的一个`inline`函数，客户将“f函数本体”编进其程序内，一旦程序库设计者决定修改f，所有用到f的程序都必须重新编译；如果f是非`inline`函数，则只需要重新链接，比重新编译的负担少很多；

将大多数`inline`限制在小型的、被频繁调用的函数上，可使得日后的调试和二进制升级更容易，使潜在的代码膨胀问题最小化；

# 31 . 将文件之间的编译依存关系降至最低

在C++中，编译器必须在编译期间直到对象的大小，从而分配足够的空间，在以下的代码中，就必须知道`Person`类的具体实现才能通过编译，因此需要`#include "Person"`，相当于两个文件之间加上了一层依存关系；

```cpp
int main()
{
	int x;
	Person p;
}
```

但假如将`Person`替换为其指针`Person*`，则编译器知道了只要分配一个指针的大小空间即可，只需要对`class Person`进行一个前向声明，从而避免了依存；

```cpp
#include <string> //标准程序库组件不应该被前向声明
#include <memory>

class PersonImpl;
class Date;
class Address;

class Person
{
public:
	Person(const std::string& name, const Date& birthday, const Address& addr);
private:
	std::shared_ptr<PersonImpl> pImpl;
}
```

在这样的设计下，`Person`完全与`Date`，`Address`等的实现分离了，那些类的任何修改都不需要`Person`文件重新编译；

1. 如果可以使用对象的引用或指针就可以完成功能，就不要直接使用对象；
2. 尽量以声明式代替定义式；
3. 为声明式和定义式提供两个不同的头文件，取代前向声明，而是直接`#include`正确的声明式头文件；

[[指向实现的指针pimpl idiom]]

## handle class

```cpp
//Person.cpp
#include "Person.h"
#include "PersonImpl.h"

Person::Person(const std::string& name, const Date& birthday, const Address& addr) : pImpl(new PersonImpl(name, birthday, addr)) {}

std::string Person::name() const
{
	return pImpl->name();
}
```

在`pimpl idiom`的设计下，`PersonImpl`为真正做事的类，`Person`只是暴露并调用其中的接口，这种`Person`类被称为“句柄类”`handle class`；

## interface class

另一种类似的设计叫做“接口类”`interface class`，令`Person`成为一个抽象基类，不带任何的成员变量，也没有构造函数，只有一个虚析构函数和一组纯虚函数；

```cpp
//Person.h

class Person
{
public:
	virtual ~Person();
	virtual std::string name() const = 0;
	virtual std::string birthday() const = 0;
	virtual std::string address() const = 0;
public:
	static std::shared_ptr<Person> create(const std::string& name, const Date& birthday, const Address& addr); //工厂模式
}
```

```cpp
//RealPerson.h

class RealPerson : public Person
{
public:
	RealPerson(const std::string& name, const Date& birthday, const Address& addr) : theName(name), theBirthday(birthday), theAddress(addr) {}

	virtual ~RealPerson() {}
	virtual std::string name() const;
	virtual std::string birthday() const;
	virtual std::string address() const;
private:
	std::string theName;
	Date theBirthday;
	Address theAddress;
}
```

```cpp
//person.cpp

std::shared_ptr<Person> Person::create(const std::string& name, const Date& birthday, const Address& addr)
{
	return std::shared_ptr<Person>(new RealPerson(name, birthday, addr));
}
```

---

`handle class`和`interface class`解除了接口和实现之间的耦合关系，从而降低了文件之间的编译依存性；但是代价是什么呢？

对于`handle class`，成员函数必须通过`pImpl`指针取得对象数据，每次访问都会增加一层间接性，消耗的内存大小也会相应提高;

对于`interface class`，每个函数都是虚函数，因此需要付出一个间接跳跃的成本，类内也会因为虚函数表和虚表指针而增加内存占用；

# 32 . 确定你的public继承是is-a关系

public继承是一种is-a关系，适用于基类身上的每一件事情一定也适用于派生类，因为每个派生类对象也是一个基类对象；

# 33 . 避免遮掩继承而来的成员名称

相同名称的变量，内层作用域的会遮掩外围作用域的，只需要名称相同不要求类型也相同；

```cpp
class Base
{
public:
	virtual void mf1() = 0;
	virtual void mf1(int x);
	virtual void mf2();
	void mf3();
	void mf3(double x);
}
class Derived : public Base
{
public:
	virtual void mf1(); //遮掩了Base::mf1()和Base::mf1(int x)
	void mf3(); //遮掩了Base::mf3()和Base::mf3(double x)
}

Derived ddd;
ddd.mf1(1); //错误
ddd.mf3(1.0); //错误
```

如果又需要同名函数又不想遮掩（会有这种需求吗？），有以下方式可以避免：
1. 使用using表达式；

```cpp
class Derived : public Base
{
public:
	using Base::mf1;
	using Base::mf3;
	virtual void mf1();
	void mf3();
}

Derived ddd;
ddd.mf1(1); //调用了Base::mf1(int x)
ddd.mf3(1.0); //调用了Base::mf3(double x)
```

2. 使用转发函数（forwarding function）；

```cpp
class Derived : public Base
{
public:
	using Base::mf1;
	using Base::mf3;
	virtual void mf1()
	{
		Base::mf1();
	}
}
```

# 34 . 区分接口继承与实现继承

```cpp
class Shape
{
public:
	virtual void draw() const = 0;
	virtual void error(const std::string& msg);
	int objectID() const;
}
```

声明一个纯虚函数，目的是让派生类只继承函数接口；

声明一个虚函数，目的是让派生类继承接口和缺省实现；
	派生类需要的话可以自己实现接口，不需要的话就使用基类的实现；

声明一个非虚函数，目的是让派生类继承接口并有一份强制实现；

# 35 . public virtual函数的替换选择

## non-virtual interface设计

[[NVI设计模式]]

## 函数指针与Strategy模式

[[函数指针]]
[[Strategy设计模式]]

比如计算每个游戏人物的健康指数：

```cpp
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter
{
public:
	//定义一个名为HealthCalcFunc的函数指针类型
	typedef int (*HealthCalcFunc)(const GameCharacter&);

	explicit GameCharacter(HealthCalcFunc func = defaultHealthCalc)
	: healthFunc(func) {}

	int healthValue() const { return healthFunc(*this); }
private:
	HealthCalcFunc healthFunc;
}
```

同一个人物类型的不同实体可以使用不同的健康计算函数，比如：

```cpp
class EvilBadGuy : public GameCharacter
{
	...
}

//两种计算方式，都可以用HealthCalcFunc类型代表其指针
int loseHealthQuickly(const GameCharacter&);
int loseHealthSlowly(const GameCharacter&);

int main()
{
	//badass1和badass2都是同一个类的实例，但使用的健康计算函数不同
	EvilBadGuy badass1(loseHealthQuickly);
	badass1.healthValue();
	
	EvilBadGuy badass2(loseHealthSlowly);
	badass2.healthValue();
	
	return 0;
}
```

换言之，这种做法使得“健康计算函数不再是GameCharacter继承体系内的成员函数”；

---

除了函数指针外，还有`std::function`这一更具灵活性的选项；

```cpp
class GameCharacter;
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter
{
public:
	//定义了一个名为HealthCalcFunc的、具有特定参数的std::function对象类型
	typedef std::function<int (const GameCharacter&)> HealthCalcFunc;
	
	explicit GameCharacter(HealthCalcFunc func = defaultHealthCalc)
	: healthFunc(func) {}

	int healthValue() const { return healthFunc(*this); }
private:
	HealthCalcFunc healthFunc;
}
```

`std::function`类型的对象拥有“可以持有任何与此签名式兼容的可调用物”的能力，即其接受隐式转换；对于上文中的`HealthCalcFunc`，它接受可以隐式转为`const GameCharacter&`类型的参数，其返回值类型也可被隐式转为`int`；

基于这些能力，可以实现以下操作：

```cpp
short calcHealth(const GameCharacter&);

struct HealthCalculator
{
	int operator()(const GameCharacter&) const {...}
};

class GameLevel
{
	float health(const GameCharacter&) const {...}
}

int main()
{
	EvilBadGuy badass1(calcHealth);
	EvilBadGuy badass2(HealthCalculator());

	GameLevel curLevel;
	EvilBadGuy badass3(std::bind(&GameLevel::health, curLevel, std::placeholders::_1));
}
```

# 36 . 绝不重新定义继承的非虚函数

# 37 . 绝不重新定义继承而来的缺省参数值

# 38 . 通过复合塑造出has-a关系

`is-a`是完全继承，`has-a`则是部分继承；

比如想要实现一个自定义的`set`，以`std::list`作为参考，如果使用`public`继承，则会出现问题（比如列表允许元素重复而集合不允许），合适的做法是复合；

```cpp
template<class T>
class MySet
{
public:
	void inert(const T& item);
	void remove(const T& item);
private:
	std::list<T> rep;
}
```

# 39 . 谨慎使用private继承

`private`继承的规则：
1. 通过`private`继承的所有成员，在派生类中会变成私有的，无论它们在基类中是公开的还是保护的；
2. 与`public`继承不同，如果派生类`private`继承基类，编译器不会自动将一个派生类对象转换为基类对象；

那么`private`继承的意义是什么呢？
	通常来说，`private`继承在设计层面上没有明确意义，其意义只涉及软件实现层面；

`private`继承的作用是“根据某物实现出”，上一条条款中说复合也是这个功能，尽可能使用复合，必要时才用`private`继承；

# 40 . 谨慎使用多重继承

多重继承比单一继承复杂，可能导致更多的歧义性，以及对`virtual`继承的需求；

[[virtual继承与菱形继承问题]]

# 41 . 了解隐式接口和编译器多态

与隐式接口`Implicit Interface`和编译器多态`Compile-time Polymorphism`相对应的是显式接口`Explicit Interface`和运行时多态`Run-time Polymorphism`；

- 显式接口：如以下代码`void doProcessing(Widget& w)`，可以在源码（比如`Widget.h`）中找到`w`的接口，这种则称为显式接口；
- 运行期多态：`Widget`中的一些函数是虚函数，`w`调用哪个函数只有在运行期间根据动态类型才知道；

```cpp
class Widget
{
public:
	Widget();
	virtual ~Widget();
	virtual std::size_t size() const;
	virtual void normalize();
	void swap(Widget& other);
}

void doProcessing(Widget& w)
{
	if (w.size() > 10 && w != someNastyWidget)
	{
		Widget temp(w);
		temp.normalize();
		temp.swap(w);
	}
}
```

在模板及泛型编程的世界中，与面向对象不同的是，显式接口和运行期多态的使用性大大降低，范反而是隐式接口和编译器多态占了上风；

- 隐式接口：在下面这种写法中，不知道`w`具体是什么类型，只知道`w`应该实现`size, normalize, swap`这些接口，这种就叫做隐式接口；
- 编译期多态：不同的模板参数会调用不同的函数；

```cpp
template<typename T>
void doProcessing(T& w)
{
	if (w.size() > 10 && w != someNastyWidget)
	{
		Widget temp(w);
		temp.normalize();
		temp.swap(w);
	}
}
```

# 42 . 了解typename的双重含义

当声明`template`时，`typename`和`class`没有任何不同；

```cpp
template<class T>
class Widget;

template<typename T>
class Widget;
```

但是有时候一定需要使用`typename`；

1. 显式声明从属名称为类型；

```cpp
template<typename T>
void print2nd(const T& con)
{
	if (con.size() >= 2)
	{
		T::const_iterator iter(con.begin());
		++iter;
		int value = *iter;
		std::cout << value;
	}
}
```

对于两个局部变量`iter`和`value`，`iter`的类型是`T::const_iterator`，实际上是什么类型取决于参数`T`，这种`template`内的名称依赖于某个`template`参数的名称，称为从属名称`dependent names`；而`value`是内置类型`int`，称为非从属名称`non-dependent names`；

[[从属名称与非从属名称]]

`T::const_iterator`可能是一个类型，也可能是一个静态成员变量，为了让编译器知道它究竟是什么，就需要用到`typename`进行显式规范；

“`typename`必须作为从属名称的前缀词”这一规则的==例外==是 —— `typename`不可以出现在基类列表内的嵌套从属名称之前，也不可在成员初始化列表中作为基类修饰符，比如：

```cpp
template<typename T>
class Derived : public Base<T>::Nested //基类列表中不允许typename
{
public:
	explicit Derived(int x)
	: Base<T>::Nested(x) //成员初始化列表不允许typename
	{
		typename Base<T>::Nested temp; //允许使用typename
		...
	}
}
```

2. 用于std::iterator_traits指明类型；

[[std-iterator_traits]]

# 43 . 学习处理模板化基类内的名称

假如使用模板来编写一个信息发送的功能，对于不同公司有不同的处理；

```cpp
class CompanyA
{
public:
	void SendClearText(const std::string& msg);
	void SendEncryptedText(const std::string& msg);
}
class CompanyB
{
public:
	void SendClearText(const std::string& msg);
	void SendEncryptedText(const std::string& msg);
}

template<typename Company>
class MsgSender
{
public:
	void SendClear()
	{
		std::string msg = "xxx";
		Company com;
		com.SendClearText(msg);
	}
	void SendSecret()
	{
		std::string msg = "xxx";
		Company com;
		com.SendEncryptedText(msg);
	}
}
```

此时想以`MsgSender`为基础构造一个派生类，新增添加log的功能；

```cpp
template<typename Company>
class LoggingMsgSender : public MsgSender<Company>
{
public:
	void SendClearMsg()
	{
		//BeforeLog
		SendClear();
		//AfterLog
	}
}
```

但是这个派生类无法通过编译，编译器找不到对应的`SendClear`；这是因为模板的具现化是在编译期间发生的，而在编译`LoggingMsgSender`时，编译器还不知道`MsgSender<Company>`具体是什么，为了解决这个问题，可以使用以下几种方法：

1. 使用`this`指针；

```cpp
void SendClearMsg()
{
	//BeforeLog
	this->SendClear(); // 使用this->来指定SendClear是从基类继承来的
	//AfterLog
}
```

2. 提前`using`声明；

```cpp
using MsgSender<Company>::SendClear; // 明确地引入SendClear
void SendClearMsg()
{
	//BeforeLog
	SendClear(); // 现在可以直接调用了
	//AfterLog
}
```

3. 明确指出基类接口；
	但是这是最不好的解法，因为这种明确调用会破坏虚函数的功能；

```cpp
void SendClearMsg()
{
	//BeforeLog
	MsgSender<Company>::SendClear();
	//AfterLog
}
```

---

假如有一个`CompanyZ`只支持加密通讯，那它就不适用于`MsgSender`，此时可以使用模板特化来解决这个问题（`template<>`表示特化是全面性的，一旦类型参数被定义为`CompanyZ`就再没有其他参数可供变化）：

[[模板特化]]

```cpp
class CompanyZ
{
public:
	void SendEncryptedText(const std::string& msg);
}

template<>
class MsgSender<CompanyZ>
{
public:
	void SendSecret();
}
```

这种特化后的`MsgSender<CompanyZ>`中没有`SendClear`的定义，这也就是模板派生类中找不到基类中接口的其中一种情况；也可以说，相比于`OOP`，模板中的继承不是那么得畅通无阻了；

# 44 . 将与参数无关的代码抽离模板

模板会生成多个类和函数，有时模板中里掺杂了与参数无关的代码，导致源码可能看起来整齐但目标码膨胀，需要知道如何避免二进制码的冗余；

## 共性与变性分析

当两个函数中有一部分具有重复性的话，一般的做法是将共通的部分提出来，让两个函数只实现不同的不部分，这就是所谓的共性与变形分析；

编写模板时，也应该用同样的思路来避免重复，但是模板中的共性往往比较隐晦；

# 45 . 运用成员函数模板接受所有兼容类型

智能指针是“行为像指针”的对象，并提供指针没有的功能，但是原始指针支持隐式转换，智能指针不支持；

```cpp
class Top;
class Middle : public Top;
class Bottom : public Middle;

//以下功能没问题

Top* pt1 = new Middle;
Top* pt2 = new Bottom;
const Top* pt3 = pt1;
```

```cpp
template<typename T>
class SmartPtr
{
public:
	explicit SmartPtr(T* realPtr);
}

//以下为希望实现的、类似原始指针的隐式转换

SmartPtr<Top> pt1 = SmartPtr<Middle>(new Middle);
SmartPtr<Top> pt2 = SmartPtr<Bottom>(new Bottom);
SmartPtr<const Top> pt3 = pt1;
```

但是同一个`template`的不同实现体之间没有什么关系，`Top`与`Middle`之间有派生关系，但是`template<Top>`和`template<Middle>`之间没有关系，编译器会将它们视为完全不同的对象；

如果希望它们之间能够隐式转换，需要明确地添加一个对应类型的构造函数，但是这是不实际的，假如之后又新增了一个派生类，那每个`template`中又得新增一个构造函数，修改将是无穷无尽的；所以需要的不是构造函数，而是一个构造模板，这个模板是一个成员函数，作用是为类生成函数；

```cpp
template<typename T>
class SmartPtr
{
public:
	template<typename U>
	SmartPtr(const SmartPtr<U>& other);
	// 实现从 SmartPtr<U> 到 SmartPtr<T> 的转换，前提是 U* 可以转换为 T*
}
```

这种写法叫做**泛化拷贝构造函数**（`Generalized Copy Constructor`）或**模板拷贝构造函数**（`Template Copy Constructor`），用于实现类型安全的转换和兼容性，特别是在设计泛型类和或模板类（智能指针、容器等）时非常有用，但同时也需要开发者确保类型转换的安全性和合理性；

对于这个例子，还需要禁止基类到派生类这种不应该存在的转换；可以通过类型特征或`SFINAE`原则来约束转换行为，也可以依赖于编译器的类型兼容性检查来实现约束；下面的代码就是利用了编译器对原始指针转换安全性的约束；

```cpp
template<typename T>
class SmartPtr
{
public:
	template<typename U>
	SmartPtr(const SmartPtr<U>& other)
	: holdPtr(other.get()) {...} //如果U*不能转为T*，则编译器会报错

	T* get() { return holdPtr; }
private:
	T* holdPtr;
}
```

在类内声明泛化拷贝构造函数并不会组织编译器生成它们自己的非泛化拷贝构造函数，如果想要控制拷贝构造的全部方面，必须同时声明泛化和非泛化版本的拷贝构造函数，相同的规则也适用于拷贝赋值操作符；

```cpp
template<typename T>
class shared_ptr
{
public:
	shared_ptr(const shared_ptr& r); //拷贝构造

	template<typename Y>
	shared_ptr(const shared_ptr<Y>& r); //泛化拷贝构造
}
```

# 46 . 需要类型转换时请为模板定义非成员函数

[[隐式转换#成员/非成员函数的隐式转换区别]]

# 47 . 使用traits获取类型信息

假如我们想要实现一个模板函数`advance`，其功能是将输入的迭代器向后移动一定的距离，但问题是迭代器有的只能单次移动，有的支持随机移动，需要能够针对不同的迭代器执行不同的操作；

[[STL迭代器#迭代器类型]]

```cpp
template<typename IterT, typename DistT>
void advance(IterT& iter, DistT& d)
{
	if (iter是一个随机访问迭代器)
		iter += d;
	else
	{
		while (d >= 0)
		{
			++iter;
			d--;
		}
	}
}
```

为此需要获取迭代器的类型，这正是`traits`的功能 —— 允许在编译期间取得某些类型信息；

[[std-iterator_traits#获取类型信息]]

`traits`并不是一个关键词，而是一个C++程序员共同遵守的协议，只要遵守这个协议，它对内置类型和用户自定义类型的表现就能一致；

# 48 . 认识模板元编程

模板元编程（`Template Meta Programming`，`TMP`）是编写基于模板的C++程序并执行于编译器的过程，其有两个伟大的能力：
1. `TMP`让某些事情变得更容易，如果没有它，某些事情会是困难甚至不可能的；
2. 由于`TMP`执行于编译器，因此可将工作从运行期转移到编译器，使得程序更高效，坏处则是编译器变长了；

[[std-iterator_traits#is_base_of与typeid]]

## TMP的hello world - 计算阶乘

`TMP`中并没有真正的循环组件，所有的循环都通过递归来完成，而`TMP`中的递归并非函数运行期常见的递归，`TMP`递归不涉及递归函数的调用，而是涉及“递归模板具现化”；

通过计算阶乘的例子来认识一下`TMP`中如何处理循环：

```cpp
template<unsigned n> //定义了一个模板参数n，其是一个无符号整数
struct Factorial
{
	enum //定义一个匿名枚举类型，有一个成员value，value是递归计算的结果
	{
		value = n * Factorial<n-1>::value
	};
}

template<> //特化版本，处理0!的特殊情况
struct Factorial<0>
{
	enum { value = 1 };
}
```

```cpp
int main() 
{
    unsigned int factorial_of_5 = Factorial<5>::value; // 在编译时计算5的阶乘
    std::cout << "Factorial of 5 is: " << factorial_of_5 << std::endl;
    return 0;
}
```

# 49 . 了解new-handler的行为

当operator new抛出异常以反映一个未获满足的内存需求前，他会先调用一个客户指定的错误处理函数，即`new-handler`；客户可以通过`set_new_handler`来指定这个错误处理函数；

```cpp
namespace std
{
	typedef void (*new_handler)();
	new_handler set_new_handler(new_handler p) throw();
}
```

```cpp
//假设使用以下函数作为new_handler
void outOfMem()
{
	std::cerr << "Out of Memory!" << std::end;
	std::abort();
}

int main()
{
	std::set_new_handler(outOfMem);
	...
}
```

当`operator new`无法满足内存申请时，它会不断调用`new-handler`函数，直到找到足够的内存；一个设计良好的`new_handler`需要有以下能力：
1. 能够释放出内存空间；
2. 如果自身无法释放出更多内存空间，但直到其他函数可以，可以通过`set_new_handler`为其他函数来替换自己；
3. 假如实在没有办法释放出更多内存，通过`set_new_handler`置空，避免重复无用的调用；
4. 抛出`bad_alloc`异常，或者直接`abort`；

原生的`std::new_handler`是全局的，不支持对于每个类的定制化服务，但用户可以自己实现类似的机制：

```cpp
class Widget
{
public:
	static std::new_handler set_new_handler(std::new_handler p) throw()
	{
		std::new_handler oldHandler = currentNewHandler;
		currentNewHandler = p;
		return oldHandler;
	}
	static void* operator new(std::size_t size) throw(std::bad_alloc);
private:
	static std::new_handler currentNewHandler;
}

std::new_handler Widget::currentNewHandler = nullptr;
```

# 50 . 了解new和delete的合理替换时机

# 51 . 编写new和delete的规则

# 52 . 自定义了new也要自定义delete

# 53 . 不要忽视编译器的警告

# 54 . 熟悉标准程序库

# 55 . 熟悉Boost
