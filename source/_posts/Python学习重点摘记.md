---
title: Python学习重点摘记
date: 2016-10-30 20:01:37
sticky: 8
categories: 
- Python
tags:
- Python
---

## 说明
该文档为学习python过程中觉得非常重要的内容(利于理解某些实现原理)，也就是作为一名pythoner，必须知道的内容，但是好记性不如烂笔头，还是记下来，定时复习还是比较好的。

**持续更新**

<!-- more -->

## 重点

1. Python模块搜索机制
   1. 程序当前目录
   2. PYTHONPATH(环境变量)目录
   3. 标准库目录

2. CPython,PyPy,Jython
   1. CPython即底层用c/c++语言实现的Python,也是我们通常所用的python解释器
   2. PyPy是使用Python实现的python解释器，速度比CPython快
   3. Jython可以让python code跑在JVM上,并可以调用java code的解释器

3. 一个项目标准文件层次结构

   ![](http://i.imgur.com/jwwhOiY.png)

4. import导入模块顺序

   1. 标准库
   2. 第三方库
   3. 本地库

5. 命名

   1. 类命名采用驼峰命名法
   2. 异常定义使用Error前缀
   3. 函数命名使用小写,并用下划线连接每个word
   4. 模块命名使用小写，word直接连接(较多)或者用下划线

6. 库API

   1. 公共API：库暴露给用户使用的API
   2. 私有API：为实现功能，库内部使用的API，函数名前面有下划线

7. 库更新时应该注意的问题

   1. 废弃的接口不要立即删除
   2. 可以在新版本中使用`warnings`对使用废弃API的程序员显示警告信息

8. pip安装包

   1. 如果在linux系统，可以使用`pip install --user 包名`来将包安装在`home`目录以避免在系统层面安装而造成操作系统目录的污染