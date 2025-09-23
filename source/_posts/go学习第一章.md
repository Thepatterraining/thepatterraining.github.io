---
title: go学习第一章
date: 2021-11-09 10:12:47
tags: ['C++']
category: C++
article: go学习第一章
---

# go学习第一章

在go现在的版本里面可以使用`go mod`来管理依赖了

使用 `go mod` 意味着不需要设置多个 `GOPATH` 了

`go mod` 对应的环境变量 `GO111MODULE` 有三个值，默认``
- on  模块支持，go命令行会使用modules，而一点也不会去GOPATH目录下查auto找
- off 无模块支持，go命令行将不会支持module功能，寻找依赖包的方式将会沿用旧版本那种通过vendor目录或者GOPATH模式来查找。
- auto 只要当前目录或者父目录有`go.mod`文件，那么就以on的形式工作。

## 标准输入输出

使用`fmt`包进行标准输出

```go
package main

import "fmt"

func main() {
    fmt.Println("hello")
}
```

使用`flag` 包进行标准输入

有两种方式
- 一种是传入变量地址的`StringVar`，第二个参数是输入的变量名，第三个是默认值，第四个是描述
- 还有一种是`String`，除了第一个参数不同，是通过返回值接收参数，其他都一样

```go
package main

import (
    "fmt"
    "flag"
)
func main() {
    var name string
    flag.StringVar(&name, "name", "default", "desc")
    name = flag.String("name","default","desc")
    flag.Parse()
    fmt.Println(name)
}
```

使用的时候`go run hello.go -name=123`

`-name`就是你输入的名称，前面要加`-` 或者 `--` 没有或者有三个都是不行的。每个输入默认自带`-h` 来获取命令帮助，帮助里面会显示描述和默认值


## go mod 下的本地包拆分

如果我们想把代码分散到不同的文件中怎么做呢？

假设现在的目录如下
- main.go
- test
    - test.go

我们想要在`main.go`引用`test.go`的代码

首先在go里面通过首字母大小写决定访问权限`首字母大写 = public`，`首字母小写 = protected or private`

假设`test.go`代码如下

```go
package test //包名和目录保持一致

func Test() {
    
}
```

`main.go`代码如下

```go
package main

func main() {

}
```

我们想使用`test`这个包里面的东西需要使用`go mod`了

执行`go mod init name`,`name` 表示当前包的名称，可以随便取

这个时候会生成`go.mod`文件，假设刚才执行的是`go mod init hello`，那么我们的`go.mod`文件如下

```
module hello

go 1.17
```

这个时候我们就可以在`main.go`引入`test.go`了，`main.go`代码如下
```go
package main

import "hello/test"  //引入hello这个包下面的test包

func main() {
    test.Test() //这里可以使用test包的Test方法了
}
```

当然了，包名`test`也可以和目录`test`不一致，比如`test.go`文件内容如下

```go
package test5 //包名和目录不一致

func Test() {
    
}
```

我们在`main.go`依然可以引入，但是需要更改一下调用，或者加个别名
```go
package main

import test5 "hello/test"  //别名，不用也可以，不过有的会报错

func main() {
    test5.Test() //这里可以使用test5包的Test方法
}
```

## 变量

变量的声明有两种方式
- var 可以使用在任何地方，不可以重复声明
    - var [name] [type]
    - type在变量声明的时候如果有赋值，那么可以省略
    - var [name] = 1
    - var 在外面的声明，可以在使用之后声明
    - var 在局部声明，必须在使用之前声明
- := 只能使用在函数等代码块里面，当有新的参数在左边声明的时候可以`重声明`

比如
```go
var a string
var b = a //b也是string
var a = "123" // 报错，因为a已经声明过了
a, c := "123","456" //不会报错，因为:=可以重声明，但是必须有一个新变量，比如c
a, c := "456", "123" //报错，因为a,c都已经声明过了，都是旧变量
```

### 简单指针变量

比如声明一个变量
```go
var a string
```

那么会产生一个内存块
|内存地址| 变量内容|
|:-----|:-----|
|001| 空|

这个时候可以通过`&`操作符取地址

```
println(a) //打印变量内容 空
println(&a) //打印变量内存地址 001
```

如果我们声明一个指针变量
```go
var b *string
b = &a  //指针变量只能存储内存地址
```

那么内存块
|内存地址| 变量内容|
|:-----|:-----|
|001| 空|
|002|001 |

这个时候输出，可以通过`*`操作符取指针的值
- 第一步先找出指针变量的内容001
- 第二步将001作为内存地址查询对应地址的内容

```go
println(b)  //输出的是变量b的内容001
println(*b) //输出的是变量b的内容001作为内存地址的内容空
println(&b) //输出的是变量b的内存地址002
```

### 变量类型转换和类型断言

可以使用 `type` 创建新的类型和声明类型的别名

```go
type astring = string  //声明一个string类型的别名
```

这个时候`astring` 和 `string` 这两个类型是完全一样的，没有任何区别

```go
var a string
func main() {
	a = "1234"
	as = a  //可以赋值
	as = astring(a) //可以类型转换
	as, ok := interface{}(a).(astring) //可以类型断言
	fmt.Println(as, ok, as == a) //可以比较
}
```

如果是创建新的类型
```go
type astring string
```

这个时候这两个类型没啥关系了，但是因为底层都是string 还是可以进行类型转换的，如果底层类型不是string，那么连转换都不行
```go
var a string
func main() {
	a = "1234"
	as = a  //不可以赋值
	as = astring(a) //可以类型转换
	as, ok := interface{}(a).(astring) //类型断言失败
	fmt.Println(as, ok, as == a) //不可以比较
}
```

#### 类型转换

类型转换，比如int8转成int16，但是这种转换只适用于int和int之间，string和string之间转换，还有上面的别名和新类型之间底层类型一致的转换
```go
var b int8
b = 1
int16(b) //直接转换
```

如果是高类型像低类型转换，那么直接取后面的位数，高位会舍弃，比如
```go
var a int16 //
a = 3000 //这个时候a在计算机存储的二进制 = 0000 1011 1011 1000‬
b := uint8(a) //如果转换成8位int，那么是取后面的8位 1011 1000‬ b = 184
```

当然了，一个int也是可以转成string的，但是会把int值当成一个`Unicode`值，如果不是一个`Unicode`能表示的，那么会显示成乱码，比如
```go
b := 69
t := string(b) //t = E
```

#### 类型断言

类型断言，想要判断一个变量是什么类型，就可以使用类型断言。使用之前需要先转成`interface`类型，`interface`是所有类型的爸爸。
返回两个值，第一个是断言并转换后的值，第二个值表示是否是这个类型，如果ok = true，那么v=转换后的值，如果ok = false, 那么v = nil（空值）

```go
var a astring
v, ok := interface{}(a).(astring)  //判断a是不是一个astring类型
```



### 数组和切片

切片的底层是一个数组，切片是对数组的引用。
- 数组 [len]string 数组长度固定不可变
- 切片 []string 切片长度可变，可以看做可变长度的数组

数组和切片都有`长度length`和`容量cap`的属性
- 数组的长度和容量都是一样的
- 切片的长度表示现在数据的长度，容量表示底层数组的长度也就是切片的最大长度

比如下面，可以看到只修改c[0]的值，但是其他的值也变了，因为是修改了底层数组a的值，所以底层数组和其他引用的值都变了。
```go
a := [3]int{1,2,3} //3长度的数组
b := a[0,2] //2长度 3容量的切片
c := a[0,1] //1长度 3容量的切片
d := b[0,1] //1长度 3容量的切片
// b c d的底层都是数组a
c[0] = 100
fmt.Println(a,b,c,d) //a [100 2 3] b [100 2] c [100] d [100]
```

#### 切片的扩容

切片的容量变化，如果切片`b`现在变成一个5长度的会怎么样呢，底层会进行一个`扩容`，会创建一个新的底层数组，然后一个新的切片，返回这个新的切片给b。

扩容以后，容量如果小于1024，每次容量会乘以2，比如b的容量3乘以2变成6，如果大于1024，那么每次会乘以1.25，但是计算完以后还会进行一个内存对齐的操作。


### 字典map

字典是一个hash表，声明方式如下,有着hash的优势，比如key-value是O(1)的复杂度，但是map是无序的，每次遍历的顺序不一定。

```go
var m1 map[int]string //key是int,value是string的map，但是这样声明的map值是nil，并且不能赋值
m1[1] = "2" //报错
m2 := map[int]int{1:1,2:2}  //key是int,value也是int的map
m3 := make(map[int]int, 5) //创建一个key是int,value也是int,长度为5的map
```

### channel

channel是一个并发安全的类型。channel分为带缓冲区的和不带缓冲区的。声明方式如下

```go
package main

import (
	"fmt"
)

func main() {
	// 示例1。
	go func() {
		fmt.Println(123)
		var ch1 chan int //里面可以传输int的channel，默认值nil，如果这样声明会造成一个永久阻塞的channel，后面的代码不会执行
		fmt.Printf("ch1:%s", ch1)
	}()

	// ch1 <- 1
    ch2 := make(chan int, 1)  //声明有一个缓冲区的channel
    ch3 := make(chan int, 0)  //声明不带缓冲区的channel
	ch2 <- 1
	// ch2 <- 2
	// m1["2"] = 3
	fmt.Println(<-ch2, ch3)
}
```

把数据传给channel,使用`ch2 <- 数据`的方式把数据传给ch2这个channel，channel的数据传递全部都是浅拷贝，下面的例子可以发现，修改s的值会使得s1的值也被修改。
```go
    ch2 := make(chan []string, 1)
	s1 := []string{"1", "2"}
	ch2 <- s1
	// ch2 <- 2
	// m1["2"] = 3
	s := <-ch2
	s[1] = "34"
	fmt.Println(s, s1)
```

数据接收使用`变量 := <- ch2` 来接收ch2的数据到一个变量中

```go
s := <- ch2
```

#### 单向channel

单向channel可以限制函数的行为，比如`chan<-`类型的只能发送数据到channel中，`<-chan`类型的只能从channel中获取数据。
```go
func getChan(ch <-chan) {
    //函数里面只能从 ch 这个 channel中获取数据而无法发送数据，这样限制了这个函数里面的行为
}

func setChan(ch chan<-) {
    //函数里面只能往 ch 这个 channel中发送数据而无法获取数据
}
```

## 函数

`go`中的函数是`一等公民`可以作为type类型，可以作为参数，可以作为返回值，可以赋值给变量,可以和nil做比较等等

函数的声明
```go
func name(arg1 int, arg2 int) (r int, err error) {
    //....
}
```

函数的类型，声明一个类型 afunc afunc的底层类型是一个接受一个string参数，返回一个int参数和一个error类型参数的函数，`函数签名`是函数的参数列表和返回值列表，如果参数列表的类型一致并且返回值列表的参数类型一致就可以认为是一样的函数。
```go
type afunc func(string) (int, error) //声明一个类型 afunc afunc的底层类型是一个接受一个string参数，返回一个int参数和一个error类型参数的函数
var a afunc //可以声明一个变量，类型是 afunc 的变量
a = name //可以把函数签名一致的函数赋值给这个函数变量
a(1,2) //等于 name(1,2)
```

## 结构体

结构体的声明
```go
type as struct{ //声明一个名称叫 as 的结构体
    a string //as 有 一个string的属性 a
    b int //一个int的属性b
}
//声明一个属于as结构体的方法String
func(this as) String() string{
    return fmt.Sprintf(this.a) //访问as的属性a
}
```

从上面的声明可以看出来，可以把struct简单的类比成class，这个as的结构体有两个属性，一个方法

使用结构体
```go
func main() {
	as1 := as{ //初始化as这个结构体
		a: "1",
		b: 1,
    }
    as2 := as{} //也可以不初始化
	fmt.Println(as1, as2)
}
```

结构体的组合，也可以类比成class的继承，不过组合比继承更有优势。组合进来以后，asT结构体就拥有了as类的属性和方法，但是由于asT有a，as也有a属性，asT的就把as的覆盖了
```go
type asT struct {
    a string //覆盖了as的a属性
    as //把as组合，嵌入进asT结构体，可以类比成 asT类继承了as类
}
as2 := asT{}
as2.a = "3" //修改的是as2的a属性
as2.as.a = "2" //修改的是as2.as.a属性 可以类比成修改了父类的a属性
as2.String() //可以调用as2.String方法，因为as有这个方法，他组合进来也拥有了这个方法
//这样可以定义asT的String方法，这样的话上面的代码就会访问这个方法了，可以类比成重写了String方法
func(this asT) String() string{
    return fmt.Sprintf(this.a) //访问asT的属性a
}
as2.as.String() //就算覆盖了，依然可以这样调用as的String方法
```

结构体可以组合多个结构体，也可以类比成多继承。但是这样有一个问题，比如组合的两个结构体内有同样名称的属性或者方法就会报错。声明的时候不会报错，只有使用的时候会报错，因为不知道使用哪个，如果指定相应的结构体进行使用就不会报错了。
```go
type asD struct{
    as
    asT
}
as3 := asD{}
as3.a := "3" //报错
as3.as.a := "4" //正常
```

还有一种方法，比如在新的结构体中定义一个同名的属性，就会覆盖其他的，所以就不会报错了

```go
type asD struct{
    a int
    as
    asT
}
as3 := asD{}
as3.a := 3 //正常
```

还有结构体中的覆盖是通过`名称`来判断覆盖的，跟数据类型没有关系，方法的覆盖也是一样，跟参数列表和返回值没有关系

结构体中指针的使用
```go
func(this asT) String() string{
    this.a = 5 //这里不可以赋值，因为this是一个值类型，这个赋值并不会真正的改变asT结构体的值，只是会改变当前this变量的值而已
    return fmt.Sprintf(this.a) //访问asT的属性a
}

func(this *asT) String() string{
    this.a = 5 //这里可以赋值，因为this是一个指针类型
    return fmt.Sprintf(this.a) //访问asT的属性a
}
```

这个是有定义的时候有区别，因为在调用的时候`go`会自动转换，比如
```go
asT.String() //如果接受的是一个*asT类型的值，这里go会转换成(&asT).String()的调用
```



## 接口

接口是`interface`和一般语言的接口没啥区别，但是go的接口是一种`无侵入式`实现，比如下面的代码我们声明了一个ai的接口，声明了一个as的结构体，这个结构体的方法和ai的方法一样，那么就算实现了ai的接口，可以赋值给ai接口类型的变量。

这里需要`方法名称`和`方法签名`这两个全部一致才算实现了这个接口，还有要注意，*as代表SetName是 *as的方法而不是as的方法，所以我们只能把&as1赋值过去，如果赋值as1会报错。因为as1只有一个GetName方法

接口变量具有三个属性
- 静态类型 ai
- 动态类型 赋值时候确定，比如赋值了&as1，那么动态类型就是*as
- 动态值 也就是&as1

```go
package main

import "fmt"

type ai interface { //声明一个名称叫 ai 的接口
	SetName(string)  //该接口有一个SetName方法
	GetName() string //有一个GetName方法
}

type as struct { //声明一个名称叫 as 的结构体
	a string //as 有 一个string的属性 a
	b int    //一个int的属性b
}

//声明一个属于as结构体的方法 GetName
func (this as) GetName() string {
	return fmt.Sprintf(this.a) //访问as的属性a
}

//声明一个属于as结构体的方法 SetName
func (this *as) SetName(name string) {
	this.a = name
}

func main() {
	var ai1 ai
	as1 := as{}
	ai1 = &as1
	fmt.Println(as1, ai1)
}
```

接口类型的nil值，接口只有声明的时候和赋值nil字面量的时候才是真正的nil值，看下面，输出结果a1是2，因为ai1不是真正的nil,ai1的动态值是nil，但是动态类型是*as，所以ai1不是nil
```go
    var as2 *as //as2是nil
	ai1 = as2
    var a1 int
	if ai1 == nil { //ai1不是nil
		a1 = 1
	} else {
		a1 = 2
	}
	fmt.Println(as1, ai1, a1)
```



## goroutine

goroutine 是一个 go的用户级线程，也叫协程。

使用的话就是下面这样 `go` 后面跟上协程需要执行的函数代码。首先启动go程序的时候，会启动一个主进程。然后主进程生成一个主线程来执行go程序的main函数。执行的时候是一个for循环。
- 执行第一次循环的时候i = 0
- 然后执行到了`go func`代码
- 由go的runtime查找是否有空闲的协程。如果没有那么创建一个协程。
- 然后把`go func`的代码放入创建好的协程。
- 最后把这个包含了`go func`代码的协程放入协程的等待队列中
- 直到有空闲的线程，从等待队列中取出一个协程，执行这个协程的代码

可以看到下面这段代码的执行结果，是什么也不会发生。
```go
func main() {
	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println(i)
		}()
	}
}
```

因为在for循环执行以后goroutine的代码还没有得到执行机会的时候，主线程main函数执行完了。那么这个时候系统的主线程就会关闭了，主进程也会关闭了。所以协程并没有执行。

看下面的代码，增加了定时器，这个时候会输出10个10，因为主线程执行到定时器的时候线程挂起，然后协程就有执行的时间了，但是协程开始执行的时候，for循环已经执行完了，这个时候变量i的值是10，所以10个协程打印出来的变量i的值都是10

```go
func main() {
	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println(i)
		}()
    }
    time.Sleep(time.Millisecond * 500)
}
```

看下面的代码，go func(i int){}(i)，增加了入参i int类型，并且在调用的时候把变量i传入了进去，那这个时候呢，执行的结果就是输出0-9的乱序，因为我们无法保证协程的执行顺序，但是由于传了当时的变量i，而go是浅拷贝，所以协程中的变量i的值被固定了。 

```go
func main() {
	for i := 0; i < 10; i++ {
		go func(i int) {
			fmt.Println(i)
		}(i)
    }
    time.Sleep(time.Millisecond * 500)
}
```

## for循环

for循环和别的语言一样
```go
for i:=0; i< 10; i++ {
    //...
}
```

还有 range 可以循环数组，切片，map。这里range会复制一个 numbers3 来进行循环，也就是说如果numbers3是数组，那么修改numbers3[0] 的值不会影响到 numbers3[0] 的值，因为数组是值类型。如果是切片那么会影响到。
```go
    numbers2 := []int{1, 2, 3, 4, 5, 6}
	numbers3 := numbers2[0:len(numbers2)]
    maxIndex2 := len(numbers2) - 1
    // i v对应key value，也可以只有一个key，没有value for i := range numbers3 {}
	for i, v := range numbers3 { //range 后面跟一个切片或者数组,map
		if i == maxIndex2 {
			numbers3[0] += v
		} else {
			numbers3[i+1] += v
        }
        v = 1 //这里的v因为是int类型，所以修改他的值不会影响到numbers3切片里面的值
	}
	fmt.Println(numbers2, numbers3)
```

## switch

switch语句不需要使用 break 了，因为go的switch只执行一个case,并且case后面可以跟多个结果，用逗号分隔，只要命中一个结果就执行这个case，所以case后面的结果也不能重复。
```go
    var a int
	switch 3 {
	case 1, 2:
		a = 1
	case 3, 4:
		a = 2
	}
	fmt.Println(a) //输出2
```

如果case后面重复，那么会报错
```go
    switch 3 {
	case 1, 2:
		a = 1
	case 2, 3, 4: //报错
		a = 2
	}
```

因为switch结果和case结果会进行判等的，所以他们两个的类型要是一样的，不然也会报错
```go
    switch 3 {
	case 1, 2:
		a = 1
	case "21", 3, 4: //报错
		a = 2
    }
    
    var a int
	var b uint8 = 3
	switch b {
	case 1, 2:
		a = 1
	case 3, 40000: //报错，因为40000 不是uint8能表示的
		a = 2
    }
    
    var b uint8 = 3
	switch 40000 {
	case 1, 2:
		a = 1
	case 3, b:  //位置换了也是报错
		a = 2
	}
```


## 