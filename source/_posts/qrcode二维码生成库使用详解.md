---
title: qrcode二维码生成库使用详解
date: 2016-10-28 15:28:06
categories: 
- Python模块学习
tags:
- Python
- qrcode
---

## 二维码简介

- [二维码(百度百科)](http://baike.baidu.com/link?url=UPraZykylcByHbMsH9RjuAWh79bgc44HkWQp9NOlrc-us7IFmpkgbQUQNmuo-T96fx9oIyp9583aKcLl_s_ZNztJKGJM1Oy3rg7vG7lJ_NDLbNAiQYH-mDyhBaZeG-dT)
- [QR Code](http://baike.baidu.com/link?url=5UKM4Ms1KH7ptZdU1P4KA_PLHDebNyXUo3nNUin_XAGOZhumuH3PuX_QMQZ5wGtEkUtHmnSkUsge1L9bRhBEdey30ZCC6qkcbbbphTj3ULTnAGhCgz9z-GiRvyUrgxc1CJcjKE09qowLuh8FISxPIa)

qrcode是二维码的一种，可以包含几千个字符。

![](https://xin053.github.io/images/wechat.png)

<!-- more -->

## qrcode使用

```bash
pip install qrcode
```

安装完之后，可以使用命令行工具:

```bash
qr "Some text" > test.png
```

也可以在python中使用`qrcode`模块:

```python
import qrcode

img = qrcode.make('Some data here')
img.save(r"C:\Users\zzx\Desktop\test.jpg")
```

## 使用 QRCode类

```python
import qrcode
qr = qrcode.QRCode(
    version=1,
    error_correction=qrcode.constants.ERROR_CORRECT_L,
    box_size=10,
    border=4,
)
qr.add_data('Some data')
qr.make(fit=True)

img = qr.make_image()
```

`version`数值从1到40，表示生成二维码的大小

`error_correction` parameter controls the error correction used for the QR Code. The following four constants are made available on the `qrcode` package:

- `ERROR_CORRECT_L`    About 7% or less errors can be corrected.


- `ERROR_CORRECT_M` (default)    About 15% or less errors can be corrected.


- `ERROR_CORRECT_Q`    About 25% or less errors can be corrected.
- `ERROR_CORRECT_H`    About 30% or less errors can be corrected.

`box_size` parameter controls how many pixels each "box" of the QR code is.

`border` parameter controls how many boxes thick the border should be (the default is 4, which is the minimum according to the specs).

**This module uses image libraries, Python Imaging Library (PIL) by default, to generate QR Codes.**

**It is recommended to use the pillow fork rather than PIL itself.**

**Pilow和PIL不兼容,默认是PIL，当你安装Pillow之后就会卸载PIL**

## 参考文档

- [qrcode github README](https://github.com/lincolnloop/python-qrcode)

