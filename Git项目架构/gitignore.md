`.gitignore`用于告诉 Git 哪些文件或目录不应该被版本控制系统跟踪；在这个文件中，你可以指定单个文件、文件类型、目录或者模式来忽略；这对于排除编译产生的文件、系统生成的文件或者某些特定的配置文件等是非常有用的；

```cpp
secret.txt //忽略某个文件

*.log //忽略某种后缀的文件

logs/ //忽略整个目录

!important.log //排除某个文件或目录

#comment //注释

**/temp/* //使用通配符，?匹配单个字符，**匹配任意中间目录
```

规则从上向下逐行应用，所以排除规则应该放在之后；