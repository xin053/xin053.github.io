---
title: Go学习笔记(三)
date: 2016-09-20 14:23:16
categories:
- Go
tags:
- Go
---

## 函数

### 函数声明

```go
func name(parameter-list) (result-list) {
	body
}
```

如果函数声明没有返回值列表，则函数返回空。

### 函数值
函数像其他值一样，拥有类型，可以被赋值给其他变量:

```go
func square(n int) int { return n * n }
func negative(n int) int { return -n }
func product(m, n int) int { return m * n }

f := square
fmt.Println(f(3)) // "9"

f = negative
fmt.Println(f(3)) // "-3"
fmt.Printf("%T\n", f) // "func(int) int"

f = product // compile error: can't assign func(int, int) int to func(int) int
```

<!-- more -->

### 匿名函数
**拥有函数名的函数只能在包级语法中被声明**，而通过函数字面量，可以绕过这一限制：

```go
// squares返回一个匿名函数。
// 该匿名函数每次被调用时都会返回下一个数的平方。
func squares() func() int {
	var x int
	return func() int {
		x++
		return x * x
	}
} 

func main() {
	f := squares()
	fmt.Println(f()) // "1"
	fmt.Println(f()) // "4"
	fmt.Println(f()) // "9"
	fmt.Println(f()) // "16"
}
```

### 可变参数
在参数列表的最后一个参数类型之前加上省略符号`...`， 这表示该函数会接收任意数量的该类型参数。

```go
func sum(vals...int) int {
	total := 0
	for _, val := range vals {
		total += val
	} 
	return total
}
```

在函数体中,`vals`被看作是类型为`[] int`的切片。如果原始参数已经是切片类型， 我们该如何传递给`sum`？只需在最后一个参数后加上省略符。

```go
values := []int{1, 2, 3, 4}
fmt.Println(sum(values...)) // "10"
```

## defer语法
在调用普通函数或方法前加上关键字defer， 就完成了defer所需要的语法。 当defer语句被执行时， 跟在defer后面的函数会被延迟执行。 **直到包含该defer语句的函数执行完毕时，defer后的函数才会被执行**， 不论包含defer语句的函数是通过return正常结束， 还是由于panic导致的异常结束。 **你可以在一个函数中执行多条defer语句， 它们的执行顺序与声明顺序相反**。

**defer语句经常被用于处理成对的操作， 如打开、 关闭、 连接、 断开连接、 加锁、 释放锁。 通过defer机制， 不论函数逻辑多复杂， 都能保证在任何执行路径下， 资源被释放。 释放资源的defer应该直接跟在请求资源的语句后。**

## panic机制
虽然Go的panic机制类似于其他语言的异常：

```go
switch s := suit(drawCard()); s {
	case "Spades": // ...
	case "Hearts": // ...
	case "Diamonds": // ...
	case "Clubs": // ...
	default:
		panic(fmt.Sprintf("invalid suit %q", s)) // Joker?
}
```

但panic的适用场景有一些不同。 由于panic会引起程序的崩溃， 因此panic一般用于严重错误， 如程序内部的逻辑不一致。 勤奋的程序员认为任何崩溃都表明代码中存在漏洞， 所以对于大部分漏洞， 我们应该使用Go提供的错误机制，而不是panic， 尽量避免程序的崩溃。 在健壮的程序中， 任何可以预料到的错误， 如不正确的输入、 错误的配置或是失败的I/O操作都应该被优雅的处理， 最好的处理方式， 就是使用Go的错误机制。

## Recover捕获异常
如果在deferred函数中调用了内置函数recover， 并且定义该defer语句的函数发生了panic异常， recover会使程序从panic中恢复， 并返回panic value。 导致panic异常的函数不会继续运行， 但能正常返回。 在未发生panic时调用recover， recover会返回nil。

```go
func Parse(input string) (s *Syntax, err error) {
	defer func() {
		if p := recover(); p != nil {
			err = fmt.Errorf("internal error: %v", p)
		}
	}()
	// ...parser...
}
```

## 雷区
### 循环中变量的作用域

```go
var rmdirs []func()
	for _, d := range tempDirs() {
	dir := d // NOTE: necessary!
	os.MkdirAll(dir, 0755) // creates parent directories too
	rmdirs = append(rmdirs, func() {
	os.RemoveAll(dir)
	})
} 

// ...do some work…
for _, rmdir := range rmdirs {
	rmdir() // clean up
}
```

你可能会感到困惑， 为什么要在循环体中用循环变量`d`赋值一个新的局部变量， 而不是像下面的代码一样直接使用循环变量`dir`。 需要注意， 下面的代码是错误的。

```go
var rmdirs []func()
for _, dir := range tempDirs() {
	os.MkdirAll(dir, 0755) // ok
	rmdirs = append(rmdirs, func() {
	os.RemoveAll(dir) // NOTE: incorrect!
	})
}
```

问题的原因在于循环变量的作用域。 在上面的程序中， for循环语句引入了新的词法块， 循环变量`dir`在这个词法块中被声明。
需要注意， **函数值中记录的是循环变量的内存地址， 而不是循环变量某一时刻的值**。 以`dir`为例，后续的迭代会不断更新`dir`的值， 当删除操作执行时， for循环已完成， `dir`中存储的值等于最后一次迭代的值。 这意味着， 每次对`os.RemoveAll`的调用删除的都是相同的目录。

**最主要原因是匿名函数和defer，go语句都会等待循环结束后，再执行函数值**

通常， 为了解决这个问题， 我们会引入一个与循环变量同名的局部变量， 作为循环变量的副本。 比如下面的变量`dir`， 虽然这看起来很奇怪， 但却很有用。

```go
for _, dir := range tempDirs() {
	dir := dir // declares inner dir, initialized to outer dir
	// ...
}
```

## 重点

- 将切片或者数组传递给可变参数函数，需要在后面加`...`
- 虽然在可变参数函数内部， `...int` 型参数的行为看起来很像切片类型， 但实际上， 可变参数函数和以切片作为参数的函数是不同的。