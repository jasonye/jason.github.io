[toc]

go语言深入:
https://docs.google.com/presentation/d/19QeSBDh8RJeNgrUHIeGw_4caOB_56DwoWtOs8Os2aHU/edit?pli=1#slide=id.p2


参考资料:https://github.com/golang/go/wiki/Articles#misc

建议多阅读：Effective_go文档。https://golang.org/doc/effective_go.html#composite_literals




# IDE和debug和各种编译选项



使用vet工具，来发现影子变量声明
go tool vet -shadow your_file.go

可以发现下面的代码声明
```
package main

import "fmt"

func main() {  
    x := 1
    fmt.Println(x)     //prints 1
    {
        fmt.Println(x) //prints 1
        x := 2
        fmt.Println(x) //prints 2
    }
    fmt.Println(x)     //prints 1 (bad if you need 2)
}
```


## Build构建选项
参考文档: [Build构建选项](https://godoc.org/go/build#hdr-Build_Constraints)
可以指定编译的版本
that lists the conditions under which a file should be included in the package. 

使用限制：
but they must appear near the top of the file, preceded only by blank lines and other line comments. These rules mean that in Go files a build constraint must appear before the package clause.

```

// +build linux,386 darwin,!cgo
corresponds to the boolean formula:

(linux AND 386) OR (darwin AND (NOT cgo))

```



在这个页面： https://golang.org/cmd/go/ 包含了全部link相关的记录

工具
```
go tool compile
go tool link  会打印link相关的选项、

go test ./ ...  //测试目录下的所有包
查看汇编代码
go tool objdump -s main.main cmd

```
go help build 查看build的相关选项、

#
go build -v -ldflags '-w -extldflags "-static" ' -o main

这里就包含了传递给link链接器相关的参数、选项、



## debug选项

$ go build -gcflags "-N -l" -o test test.go 

go run -gcflags '-m -l' example.go // 通过该标识分析变量是否逃逸到堆栈上
-gcflags "-N -l" 关闭编译器代码优化和函数内联 
-gcflags "-m"   进行逃逸分析、
* build 限制选项
参考： https://golang.org/pkg/go/build/

```
[yejunsheng@n224-019-017 ~]$ go tool compile
usage: compile [options] file.go...
  -%	debug non-static initializers
  -+	compiling runtime
  -B	disable bounds checking
  -C	disable printing of columns in error messages
  -D path
    	set relative path for local imports
  -E	debug symbol export
  -I directory
    	add directory to import search path
  -K	debug missing line numbers
  -L	show full file names in error messages
  -N	disable optimizations
  -S	print assembly listing
  -V	print version and exit
  
  ....
```



# 单元测试

*   Fuzzing

*   


    ##3. beego项目学习
    func Run(params ...string){
        
    } 
    
    github：
    go get -u github.com/go-redis/redis



# Go编码安全

*   输入校验和净化

    ```
    import "github.com/go-playground/validator/v10"
    
    // HTML输入净化
    https://github.com/microcosm-cc/bluemonday
    ```

*   

*   腾讯的Go安全指南：https://github.com/Tencent/secguide/blob/main/Go%E5%AE%89%E5%85%A8%E6%8C%87%E5%8D%97.md#1.1.2





# 1.语言基础

## 1. 变量
    1. string不能赋值为nil


1. new和make的区别

new和make用于不同类型
new：只分配内存并且设置为zero值、并不会初始化对象、
 It's a built-in function that allocates memory, but unlike its namesakes in some other languages it does not initialize the memory, it only zeros it. That is, new(T) allocates zeroed storage for a new item of type T and returns its address, a value of type *T. In Go terminology, it returns a pointer to a newly allocated zero value of type T.


* make：
The built-in function make(T, args) serves a purpose different from new(T). It creates slices, maps, and channels only, and it returns an initialized (not zeroed) value of type T (not *T).  
原因： 
The reason for the distinction is that these three types represent, under the covers, references to data structures that must be initialized before use. A slice, for example, is a three-item descriptor containing a pointer to the data (inside an array), the length, and the capacity, and until those items are initialized, the slice is nil. 




参考文档:https://golang.org/doc/effective_go.html#data

## 2. 数据类型
rune类型：

1.  数组 和 slices

go的数组长度是固定的、长度也同样是go数组的类型

slices的打散操作：
s := []int
打散操作：实现可变参数传值：s...

1. 注意，在go中，使用数组作为函数参数传递，使用的是值拷贝。而在C++中使用的是指针拷贝
2. 

**Go和C的数组的不同点**
There are major differences between the ways arrays work in Go and C. In Go,
Arrays are values. Assigning one array to another copies all the elements.
In particular, if you pass an array to a function, it will receive a copy of the array, not a pointer to it.
The size of an array is part of its type. The types [10]int and [20]int are distinct.



## 2. 循环 if for while

多了个type switch和

## switch 
switch 可以用来进行值/类型的选择。

```
package main

import "fmt"

func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}

func main() {
	do(21)
	do("hello")
	do(true)
}

```

## select 多路通信选择符号
select 用于选择多路返回值

```
package main

import "fmt"

func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}

func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
```




## map


1. struct和structTag
2. iota 常量产生子
3. new和make的区别
    new:简单的分配内存、并且不会初始化内存，只是赋值为0，返回*T
    make:只能用于创建slices、map和channels，并且返回一个有初始值（非零）的T类型。
    
## Panic和Recover

# defer的理解
The deferred call's arguments are evaluated immediately, but the function call is not executed until the surrounding function returns. 

defer语句在函数环境结束后执行.执行顺序和调用次序相反。

* defer 关键字的调用时机以及多次调用 defer 时执行顺序是如何确定的；

defer 关键字使用传值的方式传递参数时会进行预计算，导致不符合预期的结果；
defer 传入的函数不是在退出代码块的作用域时执行的，它只会在当前函数和方法返回之前被调用。

For instance, if the deferred function is a function literal and the surrounding function has named result parameters that are in scope within the literal, the deferred function may access and modify the result parameters before they are returned.

上面所说的是，如果一个被defer调用的函数是一个 function literal，也就是说是闭包或者匿名函数，并且调用 defer的函数时一个有名返回值(named result parameters)的函数，那么 defer 可以直接访问有名返回值并进行修改的。

```
// f returns 42
func f() (result int) {
    defer func() {
        result *= 7
    }()
    return 6
}

// f returns 100
func f() int {
    i := 100
    defer func() {
        i++
    }()
    return i
}
```

​    

## nil
1. The "nil" identifier can be used as the "zero value" for interfaces, functions, pointers, maps, slices, and channels. If you don't specify the variable type the compiler will fail to compile your code because it can't guess the type.

    所以下面的代码是错误的：
    
    
```

package main

func main() {  
    var x = nil //error

    _ = x
}


```

2. 可以使用nil的Slices，但是不能使用nil的map，

```
    package main

    import (
    	"fmt"
    )
    
    func main() {
    	var s []int
    	s = append(s, 1)
    	fmt.Println(s)
    
    	var m map[string]int
    	m["one"] = 1 //error
    }
```



## 函数
    函数可以作为值传递、
    函数闭包、

# 面向对象

## interface{} 
    Go没有类的概念、但是可以在类型上定义方法、
    interface{} 既有面向对象接口的作用，又有范型的作用。

interface{} 和 nil 的比较会分别判断 interface{} 的类型和数据是否同时为空。只有都为空的情况， interface{} 才等于 nil 

Go interfaces generally belong in the package that uses values of the interface type, not the package that implements those values. The implementing package should return concrete (usually pointer or struct) types: that way, new methods can be added to implementations without requiring extensive refactoring.
    
    
**问题：interface**
1. Add怎么实现任意类型求和功能？
   
2. 结构体和对象、
    对象声明、使用、
3. 怎么实现继承？
    怎么控制可以被子类访问？
     对象中成员变量用大小写区分public、private属性
    
7. 包管理
    多个文件可以属于一个包文件、 


# 3. 协程与并发控制 

## 1. 协程
协程没有任何办法中断另一个协程的执行。但是可以协程之间通信，让另外一个协程自动中止自己的执行。

panic会停止整个进程，不仅仅是当前goroutine，也就是说整个程序都会凉凉.
recover没办法捕捉子协程中的panic。而且会导致整个协程core dump。

* panic 会根据函数的调用顺序逐层传递；
    * 被 recover 捕获之后，则定制传递，当前函数自动退出（defer 本来也就是在函数执行完毕时才会调用），其他程序自动运行；
    * 未被 recover 捕捉，一直会抛出到 main 函数，然后进程 crash；
* panic 仅限于当前协程（routine），子协程抛出 panic，父协程无法捕获；
* 虽然子协程 panic 父协程无法捕获，但子协程的 panic 会导致父协程 crash（所以子协程最好捕获 panic），想要父协程获取子协程的 panic 信息，可以在子协程 recover 之后用 channel 传递出去；

1. 因为没办法支持打开的协程的数量：
所以使用sync.WaitGroup进行协程同步


1. 协程间通信

## chan
buffered 和 unbufferd chan：
ch := make(chan int)



# 4 . 并发同步机制

## 1. 几种同步机制的区别？

    Locker

###  Cond 
    :实现一个条件变量、协程等待点或者通知时间发生 
    Pool：实现了一组临时对象，可以被单独保存或者获取。协程并发访问安全的。主要用户保存分配的对象。

###   Once
 是一个对象，只能执行一次action

### 读写锁 Mutex和RWMutex   
    Mutex：互斥锁
    RWMutex：读写互斥排他锁、


​    
### WaitGroup
     用于等待一组协程结束。

1. 怎么控制协程组结束后关闭channel
    使用sync.WaitGroup()
2. 怎么控制并发的协程数量 
    采用buffered channel控制
    var sema = make(chan struct{}, 10)

```
package main

import (
	"fmt"
	"sync"
)

func main() {
        var  wg sync.WaitGroup 
 
        wg.Add(1) // wg.Add必须在外面
        go func(wg *sync.WaitGroup){ // 传递进去的必须是指针、否则会出现死锁等待的问题？copy不起作用？为什么？研究waitgroup的实现方式
             defer wg.Done() // 不要忘记这个、

            fmt.Println("1")
           
        }(&wg)
        wg.Wait()



	fmt.Println("Hello, playground")
}

```

问题：1. 为什么Wg传递进去的必须是指针、否则会出现死锁等待的问题？copy不起作用？为什么？研究waitgroup的实现方式


### 资源池 Pool的使用、







## GOPROXY设置

1.13版本后，可以设置goproxy
go env -w GOPROXY="https://goproxy.cn,https://proxy.golang.org,direct"


# 依赖注入
https://github.com/google/wire/blob/master/docs/guide.md



# GO的工具

go
cgo
cover
fix
fmt
godoc
vet

相关材料命令字、
https://rakyll.org/go-tool-flags/



#  性能优化

*   



# 原理

介绍一下GPM：



##  Channel的实现原理？

参考网页: 

通过循环链表保存缓冲的数据。

buf：是有缓冲的channel所特有的结构，用来存储缓存数据。是个循环链表.





在 Go 语言中创建新的 Channel 时，实际上创建的都是如下所示的结构：

```go
type hchan struct {
	qcount   uint
	dataqsiz uint
	buf      unsafe.Pointer
	elemsize uint16
	closed   uint32
	elemtype *_type
	sendx    uint
	recvx    uint
	recvq    waitq
	sendq    waitq

	lock mutex
}
```

Go

[`runtime.hchan`](https://draveness.me/golang/tree/runtime.hchan) 结构体中的五个字段 `qcount`、`dataqsiz`、`buf`、`sendx`、`recv` 构建底层的循环队列：

-   `qcount` — Channel 中的元素个数；
-   `dataqsiz` — Channel 中的循环队列的长度；
-   `buf` — Channel 的缓冲区数据指针；
-   `sendx` — Channel 的发送操作处理到的位置；
-   `recvx` — Channel 的接收操作处理到的位置；

除此之外，`elemsize` 和 `elemtype` 分别表示当前 Channel 能够收发的元素类型和大小；`sendq` 和 `recvq` 存储了当前 Channel 由于缓冲区空间不足而阻塞的 Goroutine 列表，这些等待队列使用双向链表 [`runtime.waitq`](https://draveness.me/golang/tree/runtime.waitq) 表示，链表中所有的元素都是 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 结构：

```go
type waitq struct {
	first *sudog
	last  *sudog
}
```

Go

[`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 表示一个在等待列表中的 Goroutine，该结构中存储了两个分别指向前后 [`runtime.sudog`](https://draveness.me/golang/tree/runtime.sudog) 的指针以构成链表。



# 包管理

GO111MODULE=on go install

## go mod 的使用

gomod命令

升级本地包：
go get -u xxx@latest

go build, go test, and other package-building commands add new dependencies to go.mod as needed.
go get : changes the required version of a dependency (or adds a new dependency).
go mod tidy: 删除不需要的包、或者添加一个新的依赖、
查看包版本号：
go list -m -versions xxx.xxx.xxx



# 参考资料

## 常见的Go开源库：

https://juejin.cn/post/7025760668594896909



go Example
https://gobyexample.com/

go连接Redis客户端文档：

https://godoc.org/github.com/go-redis/redis#Client.LPush

3. 控制每5秒钟上报一次状态
   
    

GOMAXPROCS

