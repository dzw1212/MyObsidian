# 读取

在 C++ 中，读取文件通常使用 `<fstream>` 头文件中的 `std::ifstream` 类。以下是一些常见的读取文件的方法：

## 读取整个文件到字符串

```cpp
#include <fstream>
#include <sstream>
#include <iostream>
#include <string>

int main() {
    std::ifstream file("example.txt");
    if (!file) {
        std::cerr << "Unable to open file" << std::endl;
        return 1;
    }

    std::stringstream buffer;
    buffer << file.rdbuf();
    std::string content = buffer.str();

    std::cout << content << std::endl;

    return 0;
}
```

## 按行读取文件

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::ifstream file("example.txt");
    if (!file) {
        std::cerr << "Unable to open file" << std::endl;
        return 1;
    }

    std::string line;
    while (std::getline(file, line)) {
        std::cout << line << std::endl;
    }

    return 0;
}
```

## 按字符读取文件

```cpp
#include <fstream>
#include <iostream>

int main() {
    std::ifstream file("example.txt");
    if (!file) {
        std::cerr << "Unable to open file" << std::endl;
        return 1;
    }

    char ch;
    while (file.get(ch)) {
        std::cout << ch;
    }

    return 0;
}
```

## 读取二进制文件

```cpp
#include <fstream>
#include <iostream>
#include <vector>

int main() {
    std::ifstream file("example.bin", std::ios::binary);
    if (!file) {
        std::cerr << "Unable to open file" << std::endl;
        return 1;
    }

    std::vector<char> buffer((std::istreambuf_iterator<char>(file)), std::istreambuf_iterator<char>());

    // Process the binary data in buffer
    for (char byte : buffer) {
        std::cout << std::hex << static_cast<int>(byte) << " ";
    }

    return 0;
}
```

---

- `std::ifstream`: 用于读取文件的输入文件流类。
- `file.rdbuf()`: 获取文件流的缓冲区。
- `std::getline`: 从文件中读取一行。
- `file.get`: 从文件中读取一个字符。
- `std::ios::binary`: 以二进制模式打开文件。

这些示例展示了如何使用 `std::ifstream` 类读取文本文件和二进制文件。根据你的需求选择合适的方法。

# 写入

在 C++ 中，写入文件通常使用 `<fstream>` 头文件中的 `std::ofstream` 类。以下是一些常见的写入文件的方法：

## 基本写入文件

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::ofstream file("example.txt");
    if (!file) {
        std::cerr << "Unable to open file for writing" << std::endl;
        return 1;
    }

    file << "Hello, World!" << std::endl;
    file << "This is a test." << std::endl;

    file.close();
    return 0;
}
```


## 追加写入文件

如果你想在文件末尾追加内容，可以使用 `std::ios::app` 模式：

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
    std::ofstream file("example.txt", std::ios::app);
    if (!file) {
        std::cerr << "Unable to open file for appending" << std::endl;
        return 1;
    }

    file << "Appending this line." << std::endl;

    file.close();
    return 0;
}
```


## 写入二进制文件

如果你需要写入二进制文件，可以使用 `std::ios::binary` 模式：

```cpp
#include <fstream>
#include <iostream>
#include <vector>

int main() {
    std::ofstream file("example.bin", std::ios::binary);
    if (!file) {
        std::cerr << "Unable to open file for writing in binary mode" << std::endl;
        return 1;
    }

    std::vector<char> data = {'H', 'e', 'l', 'l', 'o'};
    file.write(data.data(), data.size());

    file.close();
    return 0;
}
```

---

1. **基本写入文件**:
   - `std::ofstream file("example.txt");`: 打开一个名为 `example.txt` 的文件进行写入。如果文件不存在，会创建一个新文件。
   - `file << "Hello, World!" << std::endl;`: 将字符串写入文件。
   - `file.close();`: 关闭文件。

2. **追加写入文件**:
   - `std::ofstream file("example.txt", std::ios::app);`: 以追加模式打开文件。
   - `file << "Appending this line." << std::endl;`: 将字符串追加到文件末尾。

3. **写入二进制文件**:
   - `std::ofstream file("example.bin", std::ios::binary);`: 以二进制模式打开文件。
   - `file.write(data.data(), data.size());`: 将二进制数据写入文件。
