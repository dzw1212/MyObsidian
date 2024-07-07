*https://github.com/fdintino/pillow-avif-plugin*

使用`Pillow`库，在`Pillow`原生支持`avif`格式之前，需要借助`pillow-avif`插件；

```console
pip install pillow
pip install pillow-avif-plugin
```

以下代码遍历指定目录，转换格式为`jpg`：

```python
import os
from PIL import Image
import pillow_avif


def convert_and_delete(directory):
    # 遍历指定目录
    for filename in os.listdir(directory):
        if filename.endswith(".avif"):
            # 构建完整的文件路径
            file_path = os.path.join(directory, filename)
            # 打开 AVIF 文件
            with Image.open(file_path) as img:
                # 构建输出文件路径，替换扩展名为 .jpg
                output_path = os.path.join(directory, filename[:-5] + ".jpg")
                # 转换并保存为 JPG
                img.convert('RGB').save(output_path, 'JPEG')
            # 删除原 AVIF 文件
            os.remove(file_path)
	        

convert_and_delete(r'C:\XXX')
```