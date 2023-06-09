# 标题

```text
# 这是一级标题
## 这是二级标题
### 这是三级标题
#### 这是四级标题
##### 这是五级标题
###### 这是六级标题
```

# 目录

在文档开头处输入`[toc]`，则会根据**标题**自动生成目录；
（部分MD编辑器不支持）

# 水平分割线

```text
---
***
```

# 文字格式

## 斜体

```text
*斜体内容*
_斜体内容_
```

## 粗体

```text
**粗体内容**
__粗体内容__
```

## 斜粗体

```text
***粗斜体内容1***
___粗斜体内容2___
**_粗斜体内容3_**
__*粗斜体内容4*__
*__粗斜体内容5__*
_**粗斜体内容6**_
```

## 删除线

```text
~~删除内容~~
```
~~删除内容~~

## 下划线

```text
<u>下划线内容</u>
```
<u>下划线内容</u>

## 高亮

```text
==高亮内容==
```
==高亮内容==

# 列表

## 有序列表

```text
1. 首先<!-- (Shift + Enter) -->
   然后
2. 其次
3. 最后
```
1. 首先
   然后
2. 其次
3. 最后

## 无序列表

```text
- 啊啊啊
- 噫噫噫
- 呜呜呜
```
- 啊啊啊
- 噫噫噫
- 呜呜呜

## 任务列表

```text
- [ ] 任务1
- [ ] 任务2
- [ ] 任务3
```
- [x] 任务1
- [x] 任务2
- [ ] 任务3

# 引用

```text
>引用内容1
>>引用内容2
```
>引用内容
>>引用内容2

# 链接

## 网页链接

```text
[网页名称](网址 "提示信息")
```
[叔叔我呀](https://www.bilibili.com "可是要生气了哦")

### 链接加粗

```text
[**网页名称**](网址 "提示信息")
```
[**叔叔我呀**](https://www.bilibili.com "可是要生气了哦")

## 图像链接

### 网络图片

```text
![图像名称](地址 "提示信息")
```
![您完全不运动是吗](https://api.jikipedia.com/upload/e3269b90ab4fd53eaacf441ba70d7984_scaled.jpg "动一动吧求你了")

### 本地图片

```text
![[本地图片路径]]
```

### 修改图片大小

```text
![图片名称|宽度] （高度会根据图片比例自动调整）
![[图片名称|宽度]]
```

# 表格

```text
|表头1|表头2|表头3|
|:-|:-:|-:| （分别表示左对齐、居中对齐、右对齐）
|数据1|数据2|数据3| （格中换行使用<br>）
```
|表头1|表头2|表头3|
|:-|:-:|-:|
|123456|123456<br>789|123456|

# 代码

## 单行代码

```text
`单行代码`
```
`void Check()`

## 多行代码

````text
```代码格式
	代码块
```
````

```c
void Check()
{
	return false;
}
```

# 注释

## 单行注释

```text
<!--单行注释-->
```
<!-- 单行注释 -->

## 多行注释

```text
%%
123
456
%%
```
%%
123
456
%%

# 脚注

```text
[^脚注名]（脚注名推荐使用数字）

[^脚注名]: 脚注内容
```
智人的基因中含有部分尼安德特人的DNA[^10]

[^10]: <人类简史>

# 拓展文本格式

|效果|格式| |
|:-:|:-:|:-:|
|放大|`<big>放大内容</big>`|<big>big</big>
|缩小|`<small>缩小内容</small>`|<small>small</small>|
|彩色|`<font color=orange>橘黄色</font>`|<font color=orange>橘黄色</font>|

# 嵌入

## 嵌入音频

```text
<audio controls="controls" preload="none" src="音频链接地址"></audio>
```

## 嵌入视频

```text
<video width="600" height="420" controls>
  <source src="movie.mp4" type="video/mp4">
  <source src="movie.ogg" type="video/ogg">
  <source src="movie.webm" type="video/webm">  
</video>
```

## 嵌入页面

```text
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="http://music.163.com/outchain/player?type=2&id=33544561&auto=1&height=66"></iframe>
```

