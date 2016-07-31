---
title: click命令行工具使用详解
date: 2016-07-31 14:45:59
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

## 基本概念
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

## echo()
之所以不使用`print()`是为了兼容python2和3，当然也是可以使用`print()`的

## 嵌套命令
可以通过group来实现命令行的嵌套，也就是让一个命令行工具具有多个命令

```python
import click

@click.group()
def cli():
    click.echo('Hello world')

@cli.command()
def initdb():
    click.echo('Initialized the database')

@cli.command()
def dropdb():
    click.echo('Dropped the database')

if __name__ == '__main__':
    cli()
```

```bash
F:\cookies\python\learnPython\5_常用模块\click>python 2_group.py --help
Usage: 2_group.py [OPTIONS] COMMAND [ARGS]...

Options:
  --help  Show this message and exit.

Commands:
  dropdb
  initdb

F:\cookies\python\learnPython\5_常用模块\click>python 2_group.py initdb
Hello world
Initialized the database

F:\cookies\python\learnPython\5_常用模块\click>python 2_group.py dropdb
Hello world
Dropped the database
```

## parameter

`@click.argument()`可以给命令添加参数

```python
import click

@click.command()
@click.option('--count', default=1, help='number of greetings')
@click.argument('name')
def hello(count, name):
    for x in range(count):
        click.echo('Hello %s!' % name)

if __name__ == '__main__':
    hello()
```

```bash
F:\cookies\python\learnPython\5_常用模块\click>python 3_argument.py
Usage: 3_argument.py [OPTIONS] NAME

Error: Missing argument "name".

F:\cookies\python\learnPython\5_常用模块\click>python 3_argument.py --help
Usage: 3_argument.py [OPTIONS] NAME

Options:
  --count INTEGER  number of greetings
  --help           Show this message and exit.

F:\cookies\python\learnPython\5_常用模块\click>python 3_argument.py zzx
Hello zzx!
```

parameter包括arguments和options，arguments一般是必须的，而options一般是可选的

### parameter类型

- **`click.STRING`**：默认类型
- **`click.INT`**：int
- **`click.FLOAT`**：float
- **`click.BOOL`**：boolean
- [**`click.File`**](#File-Arguments)(mode='r', encoding=None, errors='strict', lazy=None, atomic=False)
- [**`click.Path`**](#File-Path-Arguments)(exists=False, file_okay=True, dir_okay=True, writable=False, readable=True, resolve_path=False, allow_dash=False, path_type=None)
- [**`click.Choice`**](#Choice-Options)(choices)
- [**`click.IntRange`**](#Range-Options)(min=None, max=None, clamp=False)

#### File Arguments

```python
@click.command()
@click.argument('input', type=click.File('rb'))
@click.argument('output', type=click.File('wb'))
def inout(input, output):
    while True:
        chunk = input.read(1024)
        if not chunk:
            break
        output.write(chunk)
```

#### File Path Arguments

```python
@click.command()
@click.argument('f', type=click.Path(exists=True))
def touch(f):
    click.echo(click.format_filename(f))
```

#### Choice Options

```python
@click.command()
@click.option('--hash-type', type=click.Choice(['md5', 'sha1']))
def digest(hash_type):
    click.echo(hash_type)
```

```bash
$ digest --hash-type=md5
md5

$ digest --hash-type=foo
Usage: digest [OPTIONS]

Error: Invalid value for "--hash-type": invalid choice: foo. (choose from md5, sha1)

$ digest --help
Usage: digest [OPTIONS]

Options:
  --hash-type [md5|sha1]
  --help                  Show this message and exit.
```

#### Range Options

```python
@click.command()
@click.option('--count', type=click.IntRange(0, 20, clamp=True))
@click.option('--digit', type=click.IntRange(0, 10))
def repeat(count, digit):
    click.echo(str(digit) * count)

if __name__ == '__main__':
    repeat()
```

```bash
$ repeat --count=1000 --digit=5
55555555555555555555
$ repeat --count=1000 --digit=12
Usage: repeat [OPTIONS]

Error: Invalid value for "--digit": 12 is not in the valid range of 0 to 10.
```

### Options
#### Multi Value Options

```python
@click.command()
@click.option('--pos', nargs=2, type=float)
def findme(pos):
    click.echo('%s / %s' % pos)
```

#### Tuples as Multi Value Options

```python
@click.command()
@click.option('--item', type=(unicode, int))
def putitem(item):
    click.echo('name=%s id=%d' % item)
```

#### Multiple Options

```python
@click.command()
@click.option('--message', '-m', multiple=True)
def commit(message):
    click.echo('\n'.join(message))
```

```bash
$ commit -m foo -m bar
foo
bar
```

### Arguments
#### Variadic Arguments

```python
@click.command()
@click.argument('src', nargs=-1)
@click.argument('dst', nargs=1)
def copy(src, dst):
    for fn in src:
        click.echo('move %s to folder %s' % (fn, dst))
```

`nargs=-1`表示参数数目无限制

## User Input Prompts

```python
value = click.prompt('Please enter a valid integer', type=int)

value = click.prompt('Please enter a number', default=42.0)

if click.confirm('Do you want to continue?'):
    click.echo('Well done!')
```

## 其他功能
click还有测试模块，以及输出带样式的文本等，在此就不详细介绍了

## 参考文档

- [click v6 官方文档](http://click.pocoo.org/6/)