`xml.dom.minidom` 是 Python 标准库中用于解析和处理 XML 数据的模块之一。它提供了一种轻量级的方式来处理 XML 文档。以下是一些常见的用法示例：

# 解析 XML

## 字符串格式

```python
from xml.dom.minidom import parseString

xml_data = """<?xml version="1.0"?>
<root>
    <child name="child1">Content1</child>
    <child name="child2">Content2</child>
</root>"""

# 解析 XML 字符串
dom = parseString(xml_data)

# 获取根元素
root = dom.documentElement
print("Root element:", root.tagName)
```

## 文件格式

```python
from xml.dom.minidom import parse

# 解析 XML 文件
dom = parse('example.xml')

# 获取根元素
root = dom.documentElement
print("Root element:", root.tagName)
```

# 遍历 XML 节点

```python
# 获取所有子节点
children = root.getElementsByTagName('child')

for child in children:
    # 获取属性
    name = child.getAttribute('name')
    # 获取文本内容
    content = child.firstChild.data
    print(f"Child name: {name}, Content: {content}")
```

# 创建和修改 XML 文档

```python
from xml.dom.minidom import Document

# 创建一个新的 XML 文档
doc = Document()

# 创建根元素
root = doc.createElement('root')
doc.appendChild(root)

# 创建子元素
child = doc.createElement('child')
child.setAttribute('name', 'child1')
child.appendChild(doc.createTextNode('Content1'))
root.appendChild(child)

# 打印 XML 文档
print(doc.toprettyxml(indent="  "))
```

# 保存 XML 文档

```python
# 将 XML 文档保存到文件
with open('output.xml', 'w', encoding='utf-8') as f:
    doc.writexml(f, addindent="  ", newl="\n", encoding='utf-8')
```
