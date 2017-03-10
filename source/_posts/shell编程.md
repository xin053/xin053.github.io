---
title: shell编程
date: 2017-03-10 12:38:10
categories:
- linux
tags:
- linux
- shell
---

## Hello World

```shell
#!/bin/bash
# this is a comment
echo 'Hello World！'
exit
```

文件保存为`hello.sh`,然后修改文件的权限:

```shell
$ chmod 755 hello.sh
```

最后，执行:

```shell
$ ./hello.sh
Hello World!
```

`exit`不是必须的，但是每个命令都会返回一个退出状态给父进程，成功返回0，非0值通常被认为是错误码，良好脚本都会带上`exit`，当一个脚本不带参数`exit`来结束时，脚本的退出状态由脚本中最后执行命令来决定

`echo $?`可以用来查看前一个命令的退出状态

<!-- more -->

### 赋值

使用`=`进行赋值，**并且`=`左右两边不能有空格**,获取变量值得时候在变量名前面加`$`

```shell
$ a=1 # 如果是a = 1,那么就会被解释为执行a命令,并带有'= 1'参数
$ echo $a
1
```

### 变量

```shell
hello="a b  c   d"
echo $hello  # a b c d  变量替换
echo "$hello" # a b  c   d   部分引用
echo "${hello}" # a b  c   d
echo '$hello' # $hello   全引用
```

正如所见,变量替换会去除掉空白，全引用会禁止所有特殊符号,如果只是想输出变量的值，推荐使用`"${}"`这种形式

#### bash中变量的类型

```shell
a=2334 #整形
b=${a/23/BB} #这将把b变量从整形变为string
c=${b/BB/23} #这将把c变量从string变为整形
```

所以说bash中的变量都是无类型的

#### 特殊变量

```shell
$ ./scriptname 1 2 3 4 5 6 7 8 9 10
```

`1 2 3 4 5 6 7 8 9 10`是从命令行传入的10个参数，`$0`表示脚本名称，`$1`表示第一个参数，`${10}`表示第10个参数，`$#`位置参数的个数，`$*`所有的位置参数，被作为一个单词

每一次执行`shift`命令能够将所有位置参数向前移动一个位置，而原来第一个位置的参数则被丢弃

#### 内部变量

`$BASH` - bash二进制执行文件的位置

`$FUNCNAME` - 当前函数的名字

`$GROUPS` - 当前用户属于的组

`$HOME` - 用户home目录

`$HOSTNAME` - 主机名

`$IFS` - 内部域分隔符，该变量决定bash在解释字符串时如何识别域或单词的边界

`$LINENO` - 记录它所在shell脚本中它所在行的行号

`$OSTYPE` - 系统类型

`$PPID` - 一个进程的`$PPID`就是它的父进程的pid

`$PWD` - 当前工作目录

`$SECONDS` - 这个脚本已经运行的时间

`$SHLVL` - shell层叠的层次

`$UID` - 用户id号

`$$` - 脚本自身进程pid

#### 获取变量名

```shell
${!prefix*}
${!prefix@}
```

这两个命令都可以返回以`prefix`开头的已有变量

### Here Documents

here documents是一种重定向的形式

```shell
command << token
text
token
```

这里的command是一个可以接受标准输入的命令，token是一个用来指示嵌入文本结束的字符串。上述结构就是将text的内容当作标准输入传给了command

将`<<`改为`<<-`，shell就会忽略text开头的tab字符，这样text内容就可以缩进，从而提高代码的可读性。

```shell
cat <<- _EOF_
	hello
	world
	!!!!!
_EOF_
```

常用上述方法代替`echo`输出多行内容

### 获取用户输入

使用`read`来获取用户的输入

`read a`将获取用户的输入到变量a，如果没有提供变量名，默认变量`REPLY`会包含用户输入

`read`支持以下选项

`-a array` - 把输入赋值到数组array中，从索引号0开始

`-n num` - 读取num个输入字符，而不是整行

`-p prompt` - 为输入显示提示信息

`-r` - raw modw，不会把反斜杠字符解释为转义字符

`-s` - silent mode，不会再屏幕上显示输入的文字

`-t seconds`  - 超时，seconds秒之后，如果没有输入，则返回一个非零退出状态

### 给变量指定默认值

```shell
${parameter:-word}
```

若`parameter`没有设置或者为空，展开结果为`word`，若`parameter`不为空，则展开结果是`parameter`的值

```shell
${parameter:=word}
```

若`parameter`没有设置或者为空，展开结果为`word`，并且`word`的值会赋值给`parameter`,若`parameter`不为空，则展开结果是`parameter`的值

```shell
${parameter:?word}
```

若`parameter`没有设置或者为空，这种展开导致脚本带有错误退出，并且`word`的内容会发送到标准错误，若`parameter`不为空，则展开结果是`parameter`的值

### 函数

#### 函数定义

函数定义有两种形式

```shell
function name(){
  commands
  return
}
```

或者

```shell
name(){
  commands
  return
}
```

调用函数时，只用写函数名，不用加括号，并且函数的定义要在函数调用之前

```shell
#!/bin/bash
function hello(){
  echo "Hello World!"
  return
}
hello   # 函数调用
```

#### 局部变量

在函数内部使用`local`关键字来定义局部变量

```shell
function funcname(){
  local test=1
  echo $test
  return
}
```

### if

```shell
x=5
if [ $x == 5 ]; then          # 注意[右边的空格和]左边的空格以及==两边的空格
	echo "x equals 5"
else
	echo "x dose not equals 5"
fi
```

### 判断

**涉及到判断的地方都是检测命令的退出状态码，如果是0，表示命令成功执行，也就表示当前判断的内容为真，非0则假。**

#### 文件表达式

`-d file` - file存在并且是一个目录

`-e file` - file存在

`-f file` - file存在并且是一个普通文件

`-s file` - file存在并且其长度大于0

`-r file` - file存在并且可读

`-w file` - file存在并且可写

`-x file` - file存在并且可执行

```shell
#!/bin/bash

FILE=~/.bashrc

if [ -f "$FILE" ]; then
	echo "$FILE is a file"
fi

exit
```

#### 字符串表达式

`-n string` - 字符串string的长度大于0

`-z string` - 字符串string的长度为0

`string1 == string2` - 字符串string1等于字符串string2

`string1 > string2` - string1排列在string2之后

#### 其他判断

```shell
[[ expression ]]
```

类似于`test`

```shell
string =~ regex
```

如果string匹配正则表达式regex，则返回真

### while

```shell
#!/bin/bash

count=1
while [ "${count}" -le 5 ]; do
	echo "${count}"
	count=$((count + 1))
done
echo "finished!"

exit
```

循环中可以使用`continue`和`break`

#### 循环读取数据

```shell
#!/bin/bash

while read para1 para2 para3; do
	...
done < test.txt
```

```shell
#!/bin/bash

sort -k 1,1 -k 2n test.txt | while read para1 para2 para3; do
```

`read`每次读取文本行之后将会返回退出状态码0，知道文件末尾，返回状态码非零才结束while循环

当循环终止时，循环中创建的任意变量或赋值的变量都会消失

### until

与while类似

```shell
#!/bin/bash

count=1
until [ "${count}" -gt 5 ]; do
	echo "${count}"
	count=$((count + 1))
done
echo "finished!"

exit
```

### case

```shell
read -p "Enter selection [0-3]"
case $REPLY in
	0)	echo "Program terminated."
		exit
		;;
	1)	echo "Hostname: $HOSTNAME"
		uptime
		;;
	2)	df -h
		;;
	3)	echo "Hello"
		;;
	*)	echo "Invalid entry" >&2
		exit 1
		;;
esac
```

#### 匹配模式

`a)` - 匹配单词`a`

`a|A)` - 匹配单词`a`或`A`

`[[:alpha:]]` - 若单词是一个字母字符，则匹配

`???)` - 若单词只有3个字符，则匹配

`*.txt` - 若单词以`.txt`字符结尾，则匹配

### for

```shell
for i in A B C D; do
	echo "$i"
done
```

```shell
for i in {A..D}; do
	echo "$i"
done
```

```shell
for i in cloud*.txt; do
	echo "$i"
done
```

也可以使用c语言格式:

```shell
for (( expression1; expression2; expression3 )); do
	commands
done
```

### 字符串操作

```shell
${#parameter}
```

会展开为`parameter`所包含的字符串的长度

```shell
${parameter:offset}        # 提取从offset到末尾的字符串
${parameter:offset:length} # 提取offset开始，指定长度的字符串
```

#### 子串消除

```shell
${parameter#pattern}       # 展开为删除parameter中从开头开始匹配pattern的最短字符串
${parameter##pattern}      # 展开为删除parameter中从开头开始匹配pattern的最长字符串
```

```shell
$ foo=file.txt.zip
$ echo ${foo#*.}
txt.zip
$ echo ${foo##*.}
zip
```

```shell
${parameter%pattern}
${parameter%%pattern}
```

功能与`#`和`##`类似，只是是从结尾开始匹配

#### 字符串替换

```shell
${parameter/pattern/string}  # 用string替换第一个匹配pattern的字符串
${parameter//pattern/string} # 替换掉全部匹配的
${parameter/#pattern/string} # 替换从字符串开头开始匹配的第一个字符串
${parameter/%pattern/string} # 替换从字符串结尾开始匹配的第一个字符串
```

原parameter变量值不变

#### 字符串大小写

```shell
${parameter,,}   # 把parameter的值全部展开为小写
${parameter,}    # 仅把第一个字符展开为小写
${parameter^^}   # 把parameter的值全部展开为大写
${parameter^}    # 仅把第一个字符展开为大写
```

原parameter变量值不变

### 数组

```shell
$ declare -a array  # 声明array为一个数组
$ array[0]=0
$ array[1]=1
$ echo ${array[0]}
0
$ echo ${array[1]}
1
```

#### 多值赋值

```shell
$ test=(a b c d)
$ echo ${test[0]}
a
```

#### 输出整个数组内容

```shell
$ animals=("a dog" "a cat" "a fish")
$ for i in "${animals[*]}"; do echo $i; done
a dog a cat a fish
$ for i in "${animals[@]}"; do echo $i; done
a dog
a cat
a fish
```

下标`*`和`@`可以被用来访问数组中的每一个元素

#### 关联数组

```shell
$ declare -A colors
$ colors["red"]="#ff0000"
$ colors["green"]="#00ff00"
$ colors["blue"]="#0000ff"
$ echo ${colors["blue"]}
#0000ff
```

#### 找到数组使用的下标

bash允许数组下标包含空格，有时候确定哪个元素真正存在是很有用的

```shell
${!array[*]}
${!array[@]}
```

### 组命令和子shell

组命令

```shell
{ command1; command2; [commands3; ...] }  # 注意花括号旁边的空格
```

子shell

```shell
(command1; command2; [command3; ...])
```

组命令和子shell都是用来管理重定向的

```shell
{ ls -l; echo "test"; cat foo.txt } > output.txt
```

会将三个命令的结果合成在一起然后重定向到`output.txt`中

组命令是在当前shell中执行它所有的命令，而子shell是在一个子shell中执行命令，在子shell中执行命令对环境变量等修改在子shell消失之后便会消失，大多数情况下，我们使用组命令。

```shell
$ echo "foo" | read
$ echo $REPLY
```

该`REPLY`变量的内容总是空，**是应为在管道线中的命令总是在子shell中执行的**，bash提供进程替换来解决这个问题

#### 进程替换

`<(list)` - 一种适用于产生标准输出的进程

`>(list)` - 一种适用于接受标准输入的进程

```shell
read < <(echo "foo")
echo $REPLY
```

进程替换允许我们把一个子shell的输出结果当作一个用于重定向的普通文件，事实上，它就是一种展开形式