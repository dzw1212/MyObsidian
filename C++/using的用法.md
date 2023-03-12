## 省略命名空间

```c++
using namespace std;

cout << string("Ignore std") << endl;
```

## 指定别名

```c++
template<typename T>
using EventFn = std::function<bool(T&)>;

using INT = int;

bool TestFunc(int& nVal)
{
	return nVal >= 0;
}


int main()
{
	INT i = 5;
	EventFn<int> func1 = TestFunc; //OK
	EventFn<float> func2 = TestFunc; //Nope

	return 0;
}
```

### 与typedef的区别

using支持模板类型，而typedef不支持；
```c++
template<typename T>
using func1 = std::function<bool(T)> //OK

template<typename T>
typedef std::function<bool(T)> func2; //Nope
```

## 在子类中引用基类的成员

在子类中对基类成员使用using进行声明，可以恢复该成员在基类中的保密级别；