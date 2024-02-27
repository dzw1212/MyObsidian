Git的`submodule`允许你将一个Git仓库作为另一个Git仓库的子目录，这对于包含第三方代码或库非常有用；

# .gitmodules文件

一般来说Git会自动修改这个文件，但某些时候（比如删除时）需要手动修改；

```cpp
[submodule "Dazel/Vender/spdlog"]
	path = Dazel/Vender/spdlog
	url = https://github.com/gabime/spdlog.git
[submodule "Dazel/Vender/glm"]
	path = Dazel/Vender/glm
	url = https://github.com/dzw1212/glm.git
[submodule "Dazel/Vender/GLFW"]
	path = Dazel/Vender/GLFW
	url = https://github.com/dzw1212/glfw.git
[submodule "Dazel/Vender/ImGui"]
	path = Dazel/Vender/ImGui
```

# 添加

```cpp
//添加
git submodule add <URL> path/to/submodule

//初始化
git submodule init

//更新
git submodule update
```

# 拉取

对于含有submodule的项目，可以使用以下语法来自动初始化和更新每个子模块；

```cpp
git clone --recurse-submodules <URL>
```

# 删除

1. 删除`Submodule`配置和数据
	编辑`.gitmodules`文件，删除对应的`submodule`配置；
	编辑`.git/config文件，删除对应的`submodule`配置；
	删除`.git/modules/<submodule_name>`目录；

2. 删除Submodule目录
```cpp
git rm --cached path/to/submodule
rm -rf path/to/submodule
```

3. 提交更改
```cpp
git commit -m "Removed submodule"
```