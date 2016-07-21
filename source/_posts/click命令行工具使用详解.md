---
title: click命令行工具使用详解
date: 2016-07-19 14:45:59
categories: 
- Python模块学习
tags:
- Python
- click
---

## click简介
click是类似[docopt](http://docopt.org/)的命令行工具，让我们可以快速的创建一个命令行工具，并且click能让我用尽量少的代码，并自动生成命令行帮助文档

![](http://click.pocoo.org/6/_static/click.png)

正如官网所说，click有三大特性：

- arbitrary nesting of commands
- automatic help page generation
- supports lazy loading of subcommands at runtime

<!-- more -->

## click基本使用
### click官网小例子

```python
import click

@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
def hello(count, name):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo('Hello %s!' % name)

if __name__ == '__main__':
    hello()
```

我在cmd中运行如下：

```bash
F:\cookies\python\learnPython\5_常用模块\click>python 1_基本使用.py
Your name: zzx
Hello zzx!

F:\cookies\python\learnPython\5_常用模块\click>python 1_基本使用.py --help
Usage: 1_基本使用.py [OPTIONS]

  Simple program that greets NAME for a total of COUNT times.

Options:
  --count INTEGER  Number of greetings.
  --name TEXT      The person to greet.
  --help           Show this message and exit.

F:\cookies\python\learnPython\5_常用模块\click>
```

### 基本概念
click是基于装饰器的，我们可以在方法上使用`click.command()`装饰器来将该方法变成一个命令行工具

```python
import click

@click.command()
def hello():
    click.echo('Hello World!')

if __name__ == '__main__':
    hello()
```

```bash
$ python hello.py
Hello World!

$ python hello.py --help
Usage: hello.py [OPTIONS]

Options:
  --help  Show this message and exit.
```

### echo()
之所以不使用`print()`是为了兼容python2和3，当然也是可以使用`print()`的