[toc]

# 垃圾回收的三个阶段:

    * Mark Setup - STW
    * Marking - Concurrent
    * Mark Termination - STW
  
第一阶段:打开写屏障(Write Barrier),所有的协程暂停工作.通常耗时10~30ms,垃圾回收进程需要等到所有的协程停止工作才能开始进行.而写成必须要等待函数调用的时候,才能够释放当前CPU,暂停工作.

第二阶段:标记阶段,抢占25%的CPU容量时间,执行垃圾标记工作.扫描现存的goroutines所占领的堆栈,寻找根指针,编列堆栈.垃圾回收Mark助理,帮助一起回收.

第三阶段:Mark Termination(STW)
打开写屏障,做清扫工作,并且计算下一次垃圾回收的目标. 通常60~90ms

Sweeping-Concurrent 回收结束后,进行清扫工作.  This activity occurs when application Goroutines attempt to allocate new values in heap memory. 


执行命令:
```
GODEBUG=gctrace=1 go run gc.go
```

![802d34fe971272392866ed1cb686cf42.png](evernotecid://59662A07-E013-4AF5-8108-505FA4B3C4B4/appyinxiangcom/2475354/ENResource/p2084)



# 参考runtime文档内容、

```
// 没一行显示gc时间和内容、
gc # @#s #%: #+#+# ms clock, #+#/#/#+# ms cpu, #->#-># MB, # MB goal, # P
where the fields are as follows:
	gc #        the GC number, incremented at each GC
	@#s         time in seconds since program start
	#%          percentage of time spent in GC since program start
	#+...+#     wall-clock/CPU times for the phases of the GC
	#->#-># MB  heap size at GC start, at GC end, and live heap
	# MB goal   goal heap size
	# P         number of processors used

// 相关样例:
gc 1405 @6.068s 11%: 0.058+1.2+0.083 ms clock, 0.70+2.5/1.5/0+0.99 ms cpu, 7->11->6 MB, 10 MB goal, 12 P

// General
gc 1404     : The 1404 GC run since the program started
@6.068s     : Six seconds since the program started
11%         : Eleven percent of the available CPU so far has been spent in GC

// Wall-Clock
0.058ms     : STW        : Mark Start       - Write Barrier on
1.2ms       : Concurrent : Marking
0.083ms     : STW        : Mark Termination - Write Barrier off and clean up

// CPU Time
0.70ms      : STW        : Mark Start
2.5ms       : Concurrent : Mark - Assist Time (GC performed in line with allocation)
1.5ms       : Concurrent : Mark - Background GC time
0ms         : Concurrent : Mark - Idle GC time
0.99ms      : STW        : Mark Term

// Memory
7MB         : Heap memory in-use before the Marking started
11MB        : Heap memory in-use after the Marking finished
6MB         : Heap memory marked as live after the Marking finished
10MB        : Collection goal for heap memory in-use after Marking finished

// Threads
12P         : Number of logical processors or threads used to run Goroutines
 ```
 

# 参考文档：
 https://blog.cyeam.com/golang/2016/08/18/apatternforoptimizinggo
 https://news.ycombinator.com/item?id=17882019
 https://blog.twitch.tv/en/2019/04/10/go-memory-ballast-how-i-learnt-to-stop-worrying-and-love-the-heap-26c2462549a2/
 https://github.com/golang/go/wiki/Performance
