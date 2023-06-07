
[toc]


# 安装准备工具
* 安装wrk命令:并发工具
    https://github.com/wg/wrk
* 安装graphviz 
    * brew install graphviz


## 代码中包含 
    * runtime/pprof
    * _ "net/http/pprof"  其实net/http/pprof中只是使用runtime/pprof包来进行封装了一下，并在http端口上暴露出来.
    
    Gc、goroutinue等数据统计，参考公司代码库：http://code.byted.org/gopkg/stats
    
```

```

# 优化工作：
pprof程序通过在代码中添加断点，然后程序每秒钟停顿100次，统计程序计数器中的函数样本数。从而统计到调用最多的函数。

* 执行pprof命令：profile 性能优化
    * go tool pprof http://localhost:9876/debug/pprof/profile
    * go tool pprof http://localhost:9876/debug/pprof/heap
    
    * /debug/pprof/allocs  //  A sampling of all past memory allocations
    * /debug/pprof/profile：访问这个链接会自动进行 CPU profiling，持续 30s，并生成一个文件供下载
    * /debug/pprof/block：Goroutine阻塞事件的记录。默认每发生一次阻塞事件时取样一次。 runtime.SetBlockProfileRate
    * /debug/pprof/goroutine：活跃Goroutine的信息的记录。仅在获取时取样一次。
    * /debug/pprof/heap： 堆内存分配情况的记录。默认每分配512K字节时取样一次。  检测是否有内存泄漏、 A sampling of memory allocations of live objects. You can specify the gc GET parameter to run GC before taking the heap sample
    
    * /debug/pprof/mutex: 查看争用互斥锁的持有者。 检测CPU是否没有完全利用上。默认没有打开，需要设置。runtime.SetMutexProfileFraction 打开配置、
    * 
    * /debug/pprof/threadcreate: 系统线程创建情况的记录。 仅在获取时取样一次。
* 命令： top 10 
    * 查看性能瓶颈在哪个函数 
    * 参考这个查看每列内容： https://blog.golang.org/profiling-go-programs

查看定位go的问题
go tool pprof -http :8888 http://10.153.98.142:9384/debug/pprof/heap

http://10.156.28.77:9346/debug/pprof/ 直接查看pprof数据
http://10.13.35.152:9258/debug/pprof/goroutine?debug=1

 * 每列内容展示：
    第一行：显示样本数量
    第二行：显示统计到的函数被调用样本数
    第三列：第二列的累积数据
    第四列：调用栈上的函数次数、
    第五列：调用栈上的占比、



* 命令： web 
    产生svg文件，用浏览器打开，可以查看到每次调用的瓶颈，
    如果操作系统没有使用chrome作为默认打开方式，那么可以自己设置，右击文件->现实简介->打开方式:设置chrome浏览器等.

* 命令 top5 -cum  按第四、第五列排序
* svg、gif、pdf等命令、
       框的大小与函数被调用的样本数相同、X-》Y显示函数调用关系、边上的数字：表示函数调用次数
* 命令：list DFS 显示函数内容
   
   第一列 显示该行的调用样本数、the number of samples taken while running that line 、
    第二列 正在运行或者从该行发起的调用样本数、the number of samples taken while running that line or in code called from that line
    第三列 行号
    
* 命令： disasm  显示函数汇编代码、
* 命令：weblist 显示源码/汇编代码，通过单击变换显示方式、

   

## 内存分配情况：
通过添加选项： -memprofile
`go tool pprof` 添加--inuse_objects标志，记录分配的对象次数，而不是分配的对象大小、

```
go tool pprof --inuse_objects havlak3 havlak3.mprof

```

查看堆分配的内存对象：
go tool pprof --alloc_objects http://10.156.28.77:9346/debug/pprof/heap


## goroutines泄漏
The first line tells us the total number of running goroutines. 
The following lines tell us how manygoroutines belong to specific package methods. In this example, we can see the .responseReceiver method from the Broker type in the sarama package is currently using 2 goroutines. This was the silver bullet in locating the culprit of the leak.



# go -torch使用，
    * go get -u github.com/uber/go-torch
    * git clone https://github.com/brendangregg/FlameGraph.git
    * 配置 FlameGraph 路径
    * 产生torch.svg 可以看到性能瓶颈消耗在哪里
    * go-torch is deprecated, use pprof instead，go1.11之后pprof直接支持火焰图

    * 使用方法：
     命令： go-torch http://localhost:9876/debug/pprof/profile  产生火焰图
    * https://github.com/uber-archive/go-torch
    * 


# 其他：
并发工具：https://github.com/wg/wrk

# pprof 工具的使用
## 1. 查看已经保存好的profile内容
go tool pprof profile pprof.samples.cpu.004.pb.gz

## 可以创建自己的pprof工具：
参考这个页面：https://rakyll.org/custom-profiles/ 统计，实现自己的pprof统计、比如统计打开的socket数量等
```
package blobstore

import "runtime/pprof"

var openBlobProfile = pprof.NewProfile("blobstore.Open")

// Open opens a blob, all opened blobs need
// to be closed when no longer in use.
func Open(name string) (*Blob, error) {
	blob := &Blob{name: name}
	// TODO: Initialize the blob...

	openBlobProfile.Add(blob, 2) // add the current execution stack to the profile
	return blob, nil
}

// Close closes the blob and frees the
// underlying resources.
func (b *Blob) Close() error {
	// TODO: Free other resources.
	openBlobProfile.Remove(b)
	return nil
}


// 写程序引用、
package main

import (
	"fmt"
	"math/rand"
	"net/http"
	_ "net/http/pprof" // as a side effect, registers the pprof endpoints.
	"time"

	"myproject.org/blobstore"
)

func main() {
	for i := 0; i < 1000; i++ {
		name := fmt.Sprintf("task-blob-%d", i)
		go func() {
			b, err := blobstore.Open(name)
			if err != nil {
				// TODO: Handle error.
			}
			defer b.Close()

			// TODO: Perform some wrork, write to the blob.
		}()
	}
	http.ListenAndServe("localhost:8888", nil)
}


// 然后查看pprof效果：
 go tool pprof http://localhost:8888/debug/pprof/blobstore.Open
```

## go routinue gc学习
* 使用 GODEBUG=gctrace=1 环境变量执行程序、可以查看到程序执行时候的gc情况.

GODEBUG=gctrace=1 ./myserver






# 参考文档
* pprof文档： https://blog.golang.org/profiling-go-programs
* https://software.intel.com/en-us/blogs/2014/05/10/debugging-performance-issues-in-go-programs 
    

参考文档： https://golang.org/pkg/net/http/pprof/
参考文档2: https://blog.golang.org/profiling-go-programs
查看安装火焰图等、
https://github.com/EDDYCJY/blog/blob/7f17a29ebb6d2a506601f0e9cc3275ace0aa28cb/golang/2018-09-15-Golang%20%E5%A4%A7%E6%9D%80%E5%99%A8%E4%B9%8B%E6%80%A7%E8%83%BD%E5%89%96%E6%9E%90%20PProf.md

https://blog.golang.org/laws-of-reflection

sharemem
https://golang.org/doc/codewalk/sharemem/