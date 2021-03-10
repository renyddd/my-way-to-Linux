# golang

## nil
[ref](https://tiancaiamao.gitbooks.io/go-internals/content/zh/02.4.html)

go 语言规范中定义，任一个类型未初始化时都将对应其类型的 0 值。布尔，整型，字符串分别是 false，0 和 “”；而函数、指针、slice、map、interface 都是 nil。
文档中对 nil 的解释
```go
// nil is a predeclared identifier representing the zero value for a
// pointer, channel, func, interface, map, or slice type.
var nil Type // Type must be a pointer, channel, func, interface, map, or slice type
```
如下是 channel 初始化与否的区别。
```go
func main() {
	var ch1 chan int
	ch2 := make(chan int, 0)

	fmt.Printf("%t, %t\n", ch1, ch2)

	fmt.Println(ch1, ch2)
}

/*
%!t(chan int=<nil>), %!t(chan int=0xc000064060)
<nil> 0xc000064060
*/
```
已 channel 类型来说，当其未初始化时则对应于 nil 值，初始化过的 channel 则对应栈中的一个地址指向堆中的数据。

对于 slice 来说，**空值** 与 **nil（0 值）** 是有区别的：
```go
func main() {
	var s1 []int

	var s2 = make([]int, 0)

	fmt.Println(s1, s2)
	fmt.Printf("%p, %p\n", s1, s2)

	if s1 == nil {
	}

	if s2 == nil {
	}

	fmt.Printf("%p\n", &s1)
	fmt.Printf("%p\n", &s2)
}

/*
[] []
0x0, 0x119e9e0
0xc00000c060
0xc00000c080
*/
```
slice 的空值并不是说它是一个空指针，而是说该 slice struct 中的底层数组指针域是一个空值，接着对 slice 的初始化就是为其 struct 的指针与赋值。

[ref 一篇不错的博客](https://juejin.cn/post/6895231755091968013)

## string vs []byte
```go
// string is the set of all strings of 8-bit bytes, conventionally but not
// necessarily representing UTF-8-encoded text. A string may be empty, but
// not nil. Values of string type are immutable.
```

而 byte 又是 uint8 的别名.

## unsafe.Pointer
[ref](https://www.cnblogs.com/qcrao-2018/p/10964692.html)

go 的指针不能坐数学运算；
go 不同类型的指针不能相互转换；
go 不同类型的指针不能 == 或 != 比较；
go 不同类型的指针不能相互赋值。

> unsafe 包提供了两种能力
1. 任何类型的指针可以和 unsafe.Pointer 互相转换；
2. uintptr 类型可以和 unsafe.Pointer 互相转换。

unsafe.Pointer 不能进行数学运算，只有将 unsafe.Pointer 转换为 uintptr 才可以。

还有要注意的是，uintptr 没有指针的语义，其所指的对象会被 gc 所回收，而 unsafe.Pointer 所指的对象在有用的时候不会被回收。

```go
type Man struct {
	PP *int
	Age int
}

func main() {
	a := 7
	m := Man{&a, 5}
	fmt.Println(m)

	age := *(*int)(unsafe.Pointer(uintptr(unsafe.Pointer(&m)) + uintptr(8)))

	fmt.Println(age)
}
```

## 整合
### defer
一个函数可以有多个 defer 语句，执行顺序与调用顺序相反。

### panic，recover
```go
// https://books.studygolang.com/gopl-zh/ch5/ch5-10.html
// 使用panic和recover编写一个不包含return语句但能返回一个非零值的函数。

func NonReturn(v interface{}) (result interface{}) {
	defer func() {
		if p := recover(); p != nil {
			result = p
		}
	}()

	panic(v)
}
```
### init 函数的执行顺序
[ref](https://stackoverflow.com/questions/24790175/when-is-the-init-function-run)
import --> const --> var --> init() --> main()

1. 如果一个包 import 了一个别的包，则被 import 的先被初始化；
2. 接下来是当前包 const 的初始化；
3. 当前包变量的初始化；
4. 最终是 init 函数的执行。

> A package can have multiple init functions (either in a single file or distributed across multiple files) and they are called in the order in which they are presented to the compiler.

一个包可以拥有多个 init 尽管会跨越多个文件，他们的执行顺序是按照其被呈现给编译器的顺序。
一个包也只会被初始化一次，尽管它会被多个包引用。

### context
https://juejin.cn/post/6844903555145400334


## pool
[ref 简介](https://www.jianshu.com/p/2bd41a8f2254)



## go 回收策略
[ref](https://blog.cyeam.com/golang/2017/02/08/go-optimize-slice-pool)
栈可以理解成一次函数调用，内部申请到的内存，会随着函数的返回将内存还给系统：

```go
func F() {
	temp := make([]int, 0, 20)
	...
}
```

如上的 temp 只是临时变量，并不会作为返回值返回，就被编译器划分到栈中。随着函数的返回直接释放，不会引起垃圾回收，对性能没有影响。

```go
func F() []int{
	a := make([]int, 0, 20)
	return a
}
```

申请后的内存被当作返回值返回了，不会随着函数的返回而释放，因此该内存被申请到了堆上。申请到堆上的内存才会引发垃圾回收。

```go
func F() {
	a := make([]int, 0, 20)
	b := make([]int, 0, 20000)

	l := 20
	c := make([]int, 0, l)
}
```

a 已介绍过，而 b 的内存使用较大会被转移至堆，即使它是临时变量。
c 因为它是不定长的申请，也会申请到堆上，即使它很短。


















