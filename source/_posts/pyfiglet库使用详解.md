---
title: pyfiglet库使用详解
date: 2016-07-06 16:29:12
categories: 
- Python模块学习
tags:
- Python
- pyfiglet
---

## pyfiglet简介
pyfiglet是[figlet](http://www.figlet.org/)的python实现。

那么figlet是什么?好吧，就是这个：

```
 _____ ___ ____ _      _   
|  ___|_ _/ ___| | ___| |_ 
| |_   | | |  _| |/ _ \ __|
|  _|  | | |_| | |  __/ |_ 
|_|   |___\____|_|\___|\__|
```

说简单点就是输出像上图这样的由字符拼凑的图形。

<!-- more -->

## 使用

```bash
pip install pyfiglet
```

安装好pyfiglet之后就同时安装好了pyfiglet命令行工具和pyfiglet模块。

### 使用pyfiglet命令行工具

```bash
C:\WINDOWS\system32>pyfiglet zzx

 __________  __
|_  /_  /\ \/ /
 / / / /  >  <
/___/___|/_/\_\
```

pyfiglet使用帮助：

```bash
C:\WINDOWS\system32>pyfiglet
Usage: pyfiglet [options] [text..]

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -f FONT, --font=FONT  font to render with (default: standard)
  -D DIRECTION, --direction=DIRECTION
                        set direction text will be formatted in (default:
                        auto)
  -j SIDE, --justify=SIDE
                        set justification, defaults to print direction
  -w COLS, --width=COLS
                        set terminal width for wrapping/justification
                        (default: 80)
  -r, --reverse         shows mirror image of output text
  -F, --flip            flips rendered output text over
  -l, --list_fonts      show installed fonts list
  -i, --info_font       show font's information, use with -f FONT
```

比较有用的参数也就是`-f`,`-D`,`-j`,`-r`

下面分别演示：

`-f`设置输出的字体，`-l`可以列出支持的所有字体，figlet官网有部分字体的输出预览：http://www.figlet.org/examples.html

```bash
C:\WINDOWS\system32>pyfiglet -f doh zzx

zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzxxxxxxx      xxxxxxx
z:::::::::::::::zz:::::::::::::::z x:::::x    x:::::x
z::::::::::::::z z::::::::::::::z   x:::::x  x:::::x
zzzzzzzz::::::z  zzzzzzzz::::::z     x:::::xx:::::x
      z::::::z         z::::::z       x::::::::::x
     z::::::z         z::::::z         x::::::::x
    z::::::z         z::::::z          x::::::::x
   z::::::z         z::::::z          x::::::::::x
  z::::::zzzzzzzz  z::::::zzzzzzzz   x:::::xx:::::x
 z::::::::::::::z z::::::::::::::z  x:::::x  x:::::x
z:::::::::::::::zz:::::::::::::::z x:::::x    x:::::x
zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzxxxxxxx      xxxxxxx
```

上面所有的就是`doh`这种字体的输出效果

`-D`设置图形是从左向右输出还是从右向左输出,可选参数:

- auto
- left-to-right
- right-to-left

```bash
C:\WINDOWS\system32>pyfiglet -D  left-to-right zzx

 __________  __
|_  /_  /\ \/ /
 / / / /  >  <
/___/___|/_/\_\

C:\WINDOWS\system32>pyfiglet -D right-to-left zzx

                                                                __  __ ________
                                                                \ \/ /|_  /_  /
                                                                 >  <  / / / /
                                                                /_/\_\/___/___|
```

`-j`设置输出方向，可选参数：

- auto
- left
- center
- right

```bash
C:\WINDOWS\system32>pyfiglet -j center zzx

                                 __________  __
                                |_  /_  /\ \/ /
                                 / / / /  >  <
                                /___/___|/_/\_\

C:\WINDOWS\system32>pyfiglet -j right zzx

                                                                 __________  __
                                                                |_  /_  /\ \/ /
                                                                 / / / /  >  <
                                                                /___/___|/_/\_\
```

`-r`输出字符的镜像图像

```bash
C:\WINDOWS\system32>pyfiglet -r Txt
   _      _____
 _| |_  _|_   _|
|__ \ \/ / | |
 _| |>  <  | |
|__//_/\_\ |_|
```

### 使用pyfiglet库

`Figlet`是主要的class

使用时首先要创建一个`Figlet`对象,构造方法：

```python
class Figlet(object):
    """
    Main figlet class.
    """

    def __init__(self, font=DEFAULT_FONT, direction='auto', justify='auto',
                 width=80):
        self.font = font
        self._direction = direction
        self._justify = justify
        self.width = width
        self.setFont()
        self.engine = FigletRenderingEngine(base=self)
```

参数就不再说明了，看使用：

```python
from pyfiglet import Figlet
f = Figlet(font='doh')
print(f.renderText('zzx'))

zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzxxxxxxx      xxxxxxx
z:::::::::::::::zz:::::::::::::::z x:::::x    x:::::x
z::::::::::::::z z::::::::::::::z   x:::::x  x:::::x
zzzzzzzz::::::z  zzzzzzzz::::::z     x:::::xx:::::x
      z::::::z         z::::::z       x::::::::::x
     z::::::z         z::::::z         x::::::::x
    z::::::z         z::::::z          x::::::::x
   z::::::z         z::::::z          x::::::::::x
  z::::::zzzzzzzz  z::::::zzzzzzzz   x:::::xx:::::x
 z::::::::::::::z z::::::::::::::z  x:::::x  x:::::x
z:::::::::::::::zz:::::::::::::::z x:::::x    x:::::x
zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzxxxxxxx      xxxxxxx
```

pyfiglet源码`__init__.py`中还有好几个类，感兴趣的可以去看看：

- [```__init__.py```](https://github.com/pwaller/pyfiglet/blob/master/pyfiglet/__init__.py)