---
title: Pillow图片处理库使用详解
date: 2016-10-26 20:51:34
categories: 
- Python模块学习
tags:
- Python
- Pillow
---

## Pillow 简介

是PIL的一个fork，所以不能与PIL共存

能对图片进行处理，例如转换格式，显示图片，查看图片相关信息以及resize，旋转等处理

## 使用 Image 类

```python
from PIL import Image

im = Image.open(r"C:\Users\zzx\Desktop\日语50音图.jpg")
print(im.format, im.size, im.mode)

im.show()
```

第二行获取了一个`Image`对象，第三行输出图片的格式，大小和模式

输出结果：

```bash
JPEG (650, 352) RGB
```

最后一行显示图片，官方也说明了，该`show`方法，将复制原图片为一个临时图片，然后再打开临时图片

<!-- more -->

**当使用`open`来获取一个`Image`对象时，只解析了图片的头信息以及必要的信息，而文件后面代表图片信息的数据没有解析，只有需要的时候才会去解析**

## 转换图片格式

使用`Image`对象的`save`方法对图片进行保存，文件后缀说明了文件的格式

```python
im.save(r'C:\Users\zzx\Desktop\test.png')
```

## 图片剪切

```python
box = (100, 100, 400, 400)
region = im.crop(box)
```

The region is defined by a 4-tuple, where coordinates are (left, upper, right, lower). The Python Imaging Library uses a coordinate system with (0, 0) in the upper left corner. Also note that coordinates refer to positions between the pixels, so the region in the above example is exactly 300x300 pixels.

另外paste方法可以粘贴图片

```python
region = region.transpose(Image.ROTATE_180)
im.paste(region, box)
```

## split与merge

```python
r, g, b = im.split()
im = Image.merge("RGB", (b, g, r))
```

## resize与rotate

```python
out = im.resize((128, 128))
out = im.rotate(45) # degrees counter-clockwise
```

增强图像对比度

```python
from PIL import ImageEnhance

enh = ImageEnhance.Contrast(im)
enh.enhance(1.3).show("30% more contrast")
```

## 动图处理

动图就是一帧一帧的图片

**Note that most drivers in the current version of the library only allow you to seek to the next frame (as in the above example). To rewind the file, you may have to reopen it.**

The following class lets you use the for-statement to loop over the sequence:

```python
from PIL import ImageSequence
for frame in ImageSequence.Iterator(im):
    # ...do something to frame...
```

## 图像上写字 绘图

```python
from PIL import Image
from PIL import PSDraw

im = Image.open("lena.ppm")
title = "lena"
box = (1*72, 2*72, 7*72, 10*72) # in points

ps = PSDraw.PSDraw() # default is sys.stdout
ps.begin_document(title)

# draw the image (75 dpi)
ps.image(box, im, 75)
ps.rectangle(box)

# draw title
ps.setfont("HelveticaNarrow-Bold", 36)
ps.text((3*72, 4*72), title)

ps.end_document()
```

## 从字符串获取`Image`对象

```python
import StringIO

im = Image.open(StringIO.StringIO(buffer))
```

## 从压缩文件内获取`Image`对象

```python
from PIL import TarIO

fp = TarIO.TarIO("Imaging.tar", "Imaging/test/lena.ppm")
im = Image.open(fp)
```

## 参考文档

- [Pillow官方文档](https://pillow.readthedocs.io/en/3.4.x/index.html)

