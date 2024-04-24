运行`MakeFile`文件需要`make`工具，在`Windows`上获取`make`工具的常见方法包括安装`MinGW`、`Cygwin`或使用`Windows Subsystem for Linux (WSL)`；

直接在`MakeFile`同级目录下的`console`中输入`make`即可：

```cmd
make
```

需要注意的是，`MinGW`中的`make.exe`和`Cygwin`中的`make.exe`并不相同，它们是为各自环境特别定制的。这两个环境提供了不同的工具链和目的，尽管它们都旨在提供在`Windows`上开发和编译`Unix-like`应用程序的能力；

---

假如你已经安装过`MinGW`但没有`make`包，可以通过`mingw-get`来在线获取：

```cmd
mingw-get install mingw32-make
```

