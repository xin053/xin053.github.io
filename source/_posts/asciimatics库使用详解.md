---
title: asciimatics库简介
date: 2016-07-19 11:19:03
categories: 
- Python模块学习
tags:
- Python
- asciimatics
---

## asciimatics简介
asciimatics生成ascii动画，看官方文档，上面特意强调了80后，看来那些玩过dos系统的人才能感受到终端动画的乐趣，说实话，我也只是学汇编那会儿使用cmd比较多，现在接触的比较少，这个库可以用在写命令行工具上，比较好玩，所以写下了该文章简介，不多说，看看官网介绍中利用asciimatics写的终端hello world:

![](http://i.imgur.com/uqY5mka.gif)

是不是被亮瞎了呢？

<!-- more -->

上述hello world源代码：

```python
from random import randint
from asciimatics.screen import Screen

def demo(screen):
    while True:
        screen.print_at('Hello world!',
                        randint(0, screen.width), randint(0, screen.height),
                        colour=randint(0, screen.colours - 1),
                        bg=randint(0, screen.colours - 1))
        ev = screen.get_key()
        if ev in (ord('Q'), ord('q')):
            return
        screen.refresh()

Screen.wrapper(demo)
```

asciimatics是写命令行图形界面的工具，或许以后会用得到，附上官方文档以供查阅：

[asciimatics官方文档](http://asciimatics.readthedocs.org/)