---
title: ftfy更正Unicode库使用详解
date: 2016-07-05 11:13:25
categories:
- Python模块学习
tags:
- Python
- ftfy
---

## ftfy简介
ftfy的目标是输入有问题的Unicode，输出正确的Unicode

适用于以下一些情况：

- 原本Unicode文本被用其他编码解码造成的乱码，可以通过ftfy更正
- 像html中的`&amp;`等标记会被ftfy更正
- 某些终端会带有一些控制符,如控制颜色,当复制时,就会复制这些多余的控制符
- 当从某些地方复制来的文本会出现一些显示问题，ftfy能更正

需要注意的是：输入的文本原本是Unicode，而不是其他的编码

## fix_text

```python
>>> print(fix_text('uÌˆnicode'))
ünicode

>>> print(fix_text('HTML entities &lt;3'))
HTML entities <3

>>> print(fix_text('ＬＯＵＤ　ＮＯＩＳＥＳ'))
LOUD NOISES
```

<!-- more -->

**注意：**`fix_text`每次只会处理一行，因为有可能其他行是其他的编码

当你确定整个段都是同一种编码的时候，可以使用`fix_text_segment`来代替`fix_text`,从而使整段更快的被更正。

## fix_encoding
Fix text with incorrectly-decoded garbage (“mojibake”) whenever possible.

```python
>>> print(fix_encoding('Ãºnico'))
único

>>> print(fix_encoding('The more you know ðŸŒ '))
The more you know 🌠
```

## fix_file
Fix text that is found in a file.

If the file is being read as Unicode text, use that. If it’s being read as bytes, then we hope an encoding was supplied. If not, unfortunately, we have to guess what encoding it is. We’ll try a few common encodings, but we make no promises. See the `guess_bytes` function for how this is done.

The output is a stream of fixed lines of text.

## explain_unicode
显示ftfy是如何更正的

```python
>>> explain_unicode('(╯°□°)╯︵ ┻━┻')
U+0028  (       [Ps] LEFT PARENTHESIS
U+256F  ╯       [So] BOX DRAWINGS LIGHT ARC UP AND LEFT
U+00B0  °       [So] DEGREE SIGN
U+25A1  □       [So] WHITE SQUARE
U+00B0  °       [So] DEGREE SIGN
U+0029  )       [Pe] RIGHT PARENTHESIS
U+256F  ╯       [So] BOX DRAWINGS LIGHT ARC UP AND LEFT
U+FE35  ︵      [Ps] PRESENTATION FORM FOR VERTICAL LEFT PARENTHESIS
U+0020          [Zs] SPACE
U+253B  ┻       [So] BOX DRAWINGS HEAVY UP AND HORIZONTAL
U+2501  ━       [So] BOX DRAWINGS HEAVY HORIZONTAL
U+253B  ┻       [So] BOX DRAWINGS HEAVY UP AND HORIZONTAL
```

## 命令行工具
ftfy can be used from the command line. By default, it takes UTF-8 input and writes it to UTF-8 output, fixing problems in its Unicode as it goes.

Here’s the usage documentation for the `ftfy` command:

```powershell
usage: ftfy [-h] [-o OUTPUT] [-g] [-e ENCODING] [-n NORMALIZATION]
            [--preserve-entities]
            [filename]

ftfy (fixes text for you), version 4.0.0

positional arguments:
  filename              The file whose Unicode is to be fixed. Defaults to -,
                        meaning standard input.

optional arguments:
  -h, --help            show this help message and exit
  -o OUTPUT, --output OUTPUT
                        The file to output to. Defaults to -, meaning standard
                        output.
  -g, --guess           Ask ftfy to guess the encoding of your input. This is
                        risky. Overrides -e.
  -e ENCODING, --encoding ENCODING
                        The encoding of the input. Defaults to UTF-8.
  -n NORMALIZATION, --normalization NORMALIZATION
                        The normalization of Unicode to apply. Defaults to
                        NFC. Can be "none".
  --preserve-entities   Leave HTML entities as they are. The default is to
                        decode them, as long as no HTML tags have appeared in
                        the file.
```

## ftfy.fixes模块
### ftfy.fixes.unescape_html
专门用来解码html中的标签

```python
>>> print(unescape_html('&lt;tag&gt;'))
<tag>
```

### ftfy.fixes.remove_terminal_escapes
移除终端转义符

```python
>>> print(remove_terminal_escapes(
...     "\033[36;44mI'm blue, da ba dee da ba doo...\033[0m"
... ))
I'm blue, da ba dee da ba doo...
```

### ftfy.fixes.fix_line_breaks
Convert all line breaks to Unix style.

```python
>>> def eprint(text):
...     print(text.encode('unicode-escape').decode('ascii'))

>>> eprint(fix_line_breaks("Content-type: text/plain\r\n\r\nHi."))
Content-type: text/plain\n\nHi.
```

## formatting模块
### ftfy.formatting.display_center

```python
>>> lines = ['Table flip', '(╯°□°)╯︵ ┻━┻', 'ちゃぶ台返し']
>>> for line in lines:
...     print(display_center(line, 20, '▒'))
▒▒▒▒▒Table flip▒▒▒▒▒
▒▒▒(╯°□°)╯︵ ┻━┻▒▒▒▒
▒▒▒▒ちゃぶ台返し▒▒▒▒
```

### ftfy.formatting.display_ljust

```python
>>> lines = ['Table flip', '(╯°□°)╯︵ ┻━┻', 'ちゃぶ台返し']
>>> for line in lines:
...     print(display_ljust(line, 20, '▒'))
Table flip▒▒▒▒▒▒▒▒▒▒
(╯°□°)╯︵ ┻━┻▒▒▒▒▒▒▒
ちゃぶ台返し▒▒▒▒▒▒▒▒
```

### ftfy.formatting.display_rjust

```python
>>> lines = ['Table flip', '(╯°□°)╯︵ ┻━┻', 'ちゃぶ台返し']
>>> for line in lines:
...     print(display_rjust(line, 20, '▒'))
▒▒▒▒▒▒▒▒▒▒Table flip
▒▒▒▒▒▒▒(╯°□°)╯︵ ┻━┻
▒▒▒▒▒▒▒▒ちゃぶ台返し
```

## 其他相关文章

- [ftfy官方文档](https://ftfy.readthedocs.io/en/latest/)