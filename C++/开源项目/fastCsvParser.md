一个小型、易用、快速的纯头文件库，用于读取csv文件；

*https://github.com/ben-strasser/fast-cpp-csv-parser*

# 项目配置

该库只有一个头文件`csv.h`，只需要一个符合标准的C++11编译器，没有其他依赖项；

由于使用了多线程，某些编译器需要与其他库链接才能运行，在编译参数后面添加`-lpthread`参数，比如：
```cmd
g++ -std=c++0x a.o b.o -o prog -lpthread
```

如果不希望使用多线程，可以在包含该头文件之前`#define CSV_IO_NO_THREAD`；

# 行阅读器 LineReader

用于高效地逐行读取大型文件；

```cpp
#define _CRT_SECURE_NO_WARNINGS

#include <iostream>
#include "csv.h"

int main()
{
	io::LineReader in("zh-tw.txt");
	while (const char* line = in.next_line())
	{
		std::cout << in.get_file_line() << line << std::endl;
	}

	return 0;
}
```

# CSV阅读器 CSVReader

用于高效地读取大型csv文件；

