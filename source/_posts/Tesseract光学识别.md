---
title: Tesseract光学识别
date: 2016-10-28 21:47:23
categories:
- Python模块学习
tags:
- Python
- Tesseract
- 光学识别
---

## OCR简介
本post探讨的光学识别工具是google开发的`tesseract`，项目主页：
[tesseract-ocr/tesseract](https://github.com/tesseract-ocr/tesseract)

该项目用C语言所写，官方提供了生成好的windows平台下的二进制文件，但是是老版本的，然后又说了可以从这里面[Tesseract at UB Mannheim](https://github.com/UB-Mannheim/tesseract/wiki)下到最新版的win平台打包

确实第三方维护的最新版的`tesseract`，但是我想说**安装的时候注意不要选择`PATH`那一项，作者打包时估计没注意，直接把你的`PATH`覆盖了，还好之前有备份`PATH`环境变量，要不然痛不欲生**

今天主要讲`tesseract`在win平台下的使用，以及基于`tesseract`打包的，用python写的两个库:

- [pytesseract](https://github.com/madmaze/pytesseract)
- [pyocr](https://github.com/jflesch/pyocr)

<!-- more -->

## tesseract使用

```powershell
tesseract -h
```

获取使用帮助，比较详细吧

```powershell
tesseract imagename outputbase [-l lang] [-psm pagesegmode] [configfiles...]
```

 基本就是这种用法，举个例子:

桌面有个图片，叫`1.jpg`

我们先把cmd的路径切到桌面，然后输入:

```powershell
tesseract 1.jpg test -l chi_sim
```

`-l chi_sim`表示用训练的中文数据库识别图片中的文字，不带`-l`默认使用英文，多个语言之间使用`+`连接，以下是其他语言的简写:

[LANGUAGES](https://github.com/tesseract-ocr/tesseract/blob/master/doc/tesseract.1.asc#languages)

下载其他训练好的语言数据包:

[tesseract-ocr/tessdata](tesseract-ocr/tessdata)

执行上面的命令以后，会在桌面生成一个`test.txt`文件，文件内容就是识别的文字

**评价:识别效果一般**

## pytesseract使用

之所以说是`tesseract`的封装，是因为底层就是调用上述`tesseract`命令，这个python库的使用比较简单，看下例子:

```python
from PIL import Image
import pytesseract

print(pytesseract.image_to_string(Image.open(r"C:\Users\zzx\Desktop\1.jpg")))
print(pytesseract.image_to_string(Image.open(r"C:\Users\zzx\Desktop\1.jpg"), lang='fra'))
```

直接用`print`输出到控制台可能出现乱码问题，因为`image_to_string`函数内部返回的:

```python
f = open(output_file_name)
try:
    return f.read().strip()
finally:
    f.close()
```

`open`没有指定编码，而生成的存放结果的临时文件是通过官方`tempfile`模块生成的，之后将结果写入文件是通过`subprocess.Popen`，所以临时文件中的结果与cmd的编码有关。可能会出现乱码。

## PyOCR使用

```python
from PIL import Image
import sys

import pyocr
import pyocr.builders

tools = pyocr.get_available_tools()
if len(tools) == 0:
    print("No OCR tool found")
    sys.exit(1)
# The tools are returned in the recommended order of usage
tool = tools[0]
print("Will use tool '%s'" % (tool.get_name()))
# Ex: Will use tool 'libtesseract'

langs = tool.get_available_languages()
print("Available languages: %s" % ", ".join(langs))
lang = langs[0]
print("Will use lang '%s'" % (lang))
# Ex: Will use lang 'fra'
# Note that languages are NOT sorted in any way. Please refer
# to the system locale settings for the default language
# to use.

txt = tool.image_to_string(
    Image.open('test.png'),
    lang=lang,
    builder=pyocr.builders.TextBuilder()
)

word_boxes = tool.image_to_string(
    Image.open('test.png'),
    lang="eng",
    builder=pyocr.builders.WordBoxBuilder()
)

line_and_word_boxes = tool.image_to_string(
    Image.open('test.png'), lang="fra",
    builder=pyocr.builders.LineBoxBuilder()
)

# Digits - Only Tesseract (not 'libtesseract' yet !)
digits = tool.image_to_string(
    Image.open('test-digits.png'),
    lang=lang,
    builder=pyocr.tesseract.DigitBuilder()
)
```

这是官网的例子，也很容易看懂

还能检测方向:

```python
if tool.can_detect_orientation():
    orientation = tool.detect_orientation(
        Image.open('test.png'),
        lang='fra'
    )
    pprint("Orientation: {}".format(orientation))
# Ex: Orientation: {
#   'angle': 90,
#   'confidence': 123.4,
# }
```

## 参考文档

- [tesseract-ocr/tesseract](tesseract-ocr/tesseract)
- [madmaze/pytesseract](madmaze/pytesseract)
- [jflesch/pyocr](jflesch/pyocr)

