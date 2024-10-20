*https://yaml.org/

# 特点
1. 后缀名为`.yml`或`.yaml`；
2. 相比于xml，没有了标签，更为简洁；


# 基本规则

1. 大小写敏感；
2. 使用随机表示层级关系（只能用空格，不支持Tab）；
3. 缩进的空格数不重要，只要相同层级使用相同的空格数即可；
4. 使用`#`表示注释；
5. 冒号`:`之后至少需要有1个空格作为分隔符；

# 数据结构

## 对象

键值对的集合；
```yaml
library:
	bool: Brief History of Humankind
	
# or
library : {bool: Brief History of Humankind}
```

## 数组

使用中划线`-`或中括号`[]`表示数组的每个元素；
```yaml
bool:
	- FirstBook
	- SecondBook

# or
book : [FirstBook, SecondBook]
```

## 字面量

值单个的、不可拆分的值，比如数组、字符串、布尔值等；
```yaml
name: 'd \n z \n w' # 单引号不识别转义字符，双引号可以
```

# 组织结构

一个yaml文件可以有一个或者多个文档组成，文档之间使用`---`作为分割符，每个文档之间相互独立互不干扰；
如果一个yaml文件中仅有一个文档，则分隔符可以忽略；
