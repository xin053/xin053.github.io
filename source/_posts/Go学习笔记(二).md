---
title: Go学习笔记(二)
date: 2016-09-08 19:41:42
categories:
- Go
tags:
- Go
---

## 复合数据类型
主要有四种复合数据类型：数组、slice、map和结构体。数组和结构体是具有固定大小的数据结构，slice和map是动态的数据结构，它们将根据需要动态增长。

### 数组
数组默认会生成`0~数组长度-1`的索引，于是便可以通过下标索引到数组对应元素值，也可以指定一个索引和对应值列表的方式初始化:

```go
type Currency int

const (
	USD Currency = iota 	// 美元
	EUR 			// 欧元
	GBP 			// 英镑
	RMB 			// 人民币
)

symbol := [...]string{USD: "$", EUR: "€", GBP: "£", RMB: "¥"}

fmt.Println(RMB, symbol[RMB]) // "3 ¥"
```

在这种形式的数组字面值形式中， 初始化索引的顺序是无关紧要的， 而且没用到的索引可以省略， 和前面提到的规则一样， 未指定初始值的元素将用零值初始化。

**数组在实际编程中其实用的很少.**

<!-- more -->

### Slice
Slice代表变成的序列,序列中每个元素具有相同的类型。slice的语法与数组很像，只是没有固定长度。

```go
letters := []string{"a", "b", "c", "d"}
```

如果是数组的话：

```go
letters := [...]string{"a", "b", "c", "d"}
```

**Slice底层是靠数组实现的。**一个slice是一个轻量级的数据结构，包括三个部分：指针、长度、容量。

指针指向第一个slice元素对应的底层数组元素的地址， 要注意的是slice的第一个元素并不一定就是数组的第一个元素。 长度对应slice中元素的数目；长度不能超过容量， 容量一般是从slice的开始位置到底层数据的结尾位置。 内置的`len`和`cap`函数分别返回slice的长度和容量。

多个slice之间可以共享底层的数据， 并且引用的数组部分区间可能重叠。

我们首先定义一个数组：

```go
months := [...]string{1: "January", /* ... */, 12: "December"}
```

我们声明数组时直接跳过第0个元素， 第0个元素会被自动初始化为空字符串。

![](http://i.imgur.com/IQKcgEU.png)

```go
Q2 := months[4:7]
summer := months[6:9]
fmt.Println(Q2) 	// ["April" "May" "June"]
fmt.Println(summer) 	// ["June" "July" "August"]
```

如果切片操作超出`cap(s)`的上限将导致一个`panic`异常， 但是超出`len(s)`则是意味着扩展slice， 因为新slice的长度会变大：

```go
fmt.Println(summer[:20]) 	// panic: out of range

endlessSummer := summer[:5] 	// extend a slice (within capacity)
fmt.Println(endlessSummer) 	// "[June July August September October]"
```

内置的`make`函数创建一个指定元素类型、 长度和容量的slice。 容量部分可以省略， 在这种情况下， 容量将等于长度。

```go
make([]T, len)
make([]T, len, cap) // same as make([]T, cap)[:len]
```

内置的`append`函数用于向slice追加元素：

```go
var runes []rune
for _, r := range "Hello, 世界" {
	runes = append(runes, r)
} 
fmt.Printf("%q\n", runes) // "['H' 'e' 'l' 'l' 'o' ',' ' ' '世' '界']"
```

### Map
map类型可以写为`map[K]V`， 其中`K`和`V`分别对应key和value。 map中所有的key都有相同的类型， 所有的value也有着相同的类型， 但是key和value之间可以是不同的数据类型。 **其中`K`对应的key必须是支持`==`比较运算符的数据类型，** 所以map可以通过测试key是否相等来判断是否已经存在。 

内置的`make`函数可以创建一个map：

```go
ages := make(map[string]int) // mapping from strings to ints
```

我们也可以用`map`字面值的语法创建map， 同时还可以指定一些最初的key/value：

```go
ages := map[string]int{
	"alice":   31,
	"charlie": 34,
}
```

这相当于；

```go
ages := make(map[string]int)
ages["alice"] = 31
ages["charlie"] = 34
```

因此， 另一种创建空的map的表达式是 `map[string]int{}` 。

要想遍历map中全部的key/value对的话， 可以使用`range`风格的for循环实现:

```go
for name, age := range ages {
	fmt.Printf("%s\t%d\n", name, age)
}
```

map下标语法也可以返回两个值:

```go
if age, ok := ages["bob"]; !ok { /* ... */ }
```

在这种场景下， map的下标语法将产生两个值；第二个是一个布尔值， 用于报告元素是否真的存在。 布尔变量一般命名为ok， 特别适合马上用于if条件判断部分。

### 结构体
例子:

```go
type Employee struct {
	ID 		int
	Name 		string
	Address 	string
	DoB 		time.Time
	Position 	string
	Salary 		int
	ManagerID 	int
}

var dilbert Employee
```

通常一行对应一个结构体成员， 成员的名字在前类型在后， 不过如果相邻的成员类型如果相同的话可以被合并到一行， 就像下面的`Name`和`Address`成员那样：

```go
type Employee struct {
	ID 		int
	Name, Address 	string
	DoB 		time.Time
	Position 	string
	Salary 		int
	ManagerID 	int
}
```

结构体值也可以用结构体面值表示， 结构体面值可以指定每个成员的值。

```go
type Point struct{ X, Y int }

p := Point{1, 2}
```

上面是初始化结构体的一种方式，但是这要求按顺序初始化结构体中的全部字段，另外一种方法是，**以成员名字和相应的值来初始化， 这样可以包含部分或全部的成员,这也是常用的方式.**

Go语言的结构体支持匿名成员，匿名成员的数据类型必须是命名的类型或指向一个命名的类型的指针。 

```go
type Circle struct {
	Radius int
}
 
type Wheel struct {
	Circle
	Spokes int
}
```

得意于匿名嵌入的特性， 我们可以直接访问叶子属性而不需要给出完整的路径：

```go
var w Wheel
w.Radius = 5 // equivalent to w.Circle.Radius = 5
```

## 编组
将结构体slice转为JSON的过程叫编组(marshaling)

我们首先定义一个`Movie`结构体，然后创建数组：

```go
type Movie struct {
	Title 	string
	Year 	int `json:"released"`
	Color 	bool `json:"color,omitempty"`
	Actors 	[]string
} 

var movies = []Movie{
	{Title: "Casablanca", Year: 1942, Color: false,
		Actors: []string{"Humphrey Bogart", "Ingrid Bergman"}},
	{Title: "Cool Hand Luke", Year: 1967, Color: true,
		Actors: []string{"Paul Newman"}},
	{Title: "Bullitt", Year: 1968, Color: true,
		Actors: []string{"Steve McQueen", "Jacqueline Bisset"}},
	// ...
}
```

```go
data, err := json.MarshalIndent(movies, "", " ")
if err != nil {
	log.Fatalf("JSON marshaling failed: %s", err)
} 
fmt.Printf("%s\n", data)
```

将会产生以下输出:

```go
[
	{
		"Title": "Casablanca",
		"released": 1942,
		"Actors": [
			"Humphrey Bogart",
			"Ingrid Bergman"
		]
	},
	{
		"Title": "Cool Hand Luke",
		"released": 1967,
		"color": true,
		"Actors": [
			"Paul Newman"
		]
	},
	{
		"Title": "Bullitt",
		"released": 1968,
		"color": true,
		"Actors": [
			"Steve McQueen",
			"Jacqueline Bisset"
		]
	}
]
```

**只有导出的结构体成员才会被编码， 这也就是我们为什么选择用大写字母开头的成员名称。**

可以看到， 其中`Year`名字的成员在编码后变成了`released`， 还有`Color`成员编码后变成了小写字母开头的`color`。 这是因为构体成员Tag所导致的。 一个构体成员Tag是和在编译阶段关联到该成员的元信息字符串：

```go
Year 	int 	`json:"released"`
Color 	bool 	`json:"color,omitempty"`
```

成员Tag中json对应值的第一部分用于指定JSON对象的名字.`Color`成员的Tag还带了一个额外的`omitempty`选项， 表示当Go语言结构体成员为空或零值时不生成JSON对象(这里false为零值)。

## 解组
编组的逆操作，需要先定义合适的结构体，然后使用`json.Unmarshal()`函数对json数据进行解组，具体就不介绍了。
 

## 重点

- `x[m:n]`切片操作对于字符串则生成一个新字符串
- 一个零值的slice等于`nil`。 一个`nil`值的slice并没有底层数组。 一个`nil`值的slice的长度和容量都是0， 但是也有非`nil`值的slice的长度和容量也是0的， 例如`[]int{}`或`make([]int, 3)[3:]`。
- 如果你需要测试一个slice是否是空的， 使用`len(s) == 0`来判断， 而不应该用`s == nil`来判断。