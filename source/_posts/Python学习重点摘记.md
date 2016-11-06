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