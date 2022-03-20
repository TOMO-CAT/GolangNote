# 并发与goroutine

## 并发与并行

#### 1. 并发的概念

并发意味着程序在执行时有多个执行上下文，对应着多个调用栈。每一个进程在运行时都有自己的堆和栈，有一个完整的上下文，而操作系统在调度进程的时候，会保存被调度进程的上下文环境，等该进程获得时间片后，再恢复该进程的上下文到系统中。

#### 2. 并发的场景

并发编程的常用场景包括：

* 灵敏响应的图形用户界面：用户进行操作后程序需要在后台执行复杂运算或者IO操作，为了不让用户感觉到程序的卡顿，我们需要让界面响应和后台运算同时运行
* 高并发web服务：对于每个用户请求都有单独的线程进行响应
* 计算机CPU从单内核发展成多内核，并发程序可以提高对计算机硬件的利用率
* 由于IO操作耗时高，串行编程下IO操作会导致程序阻塞

总结以下并发的优势在于：

* 客观表示问题模型：例如高并发web服务
* 利用CPU多内核优势：提高硬件利用率
* 利用CPU与其他硬件设备固有的异步性：防止IO操作导致的程序阻塞

#### 3. 并发的实现模型

| 并发模型                | 描述                                                         | 优势                                                         | 劣势                                                         |
| ----------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 多进程                  | 在操作系统层面进行并发的基本模式                             | 简单、进程间互不影响                                         | 系统开销大（所有进程都是由内核管理的）                       |
| 多线程                  | 在大部分操作系统上都属于系统层面的并发模式                   | 比多进程的开销小                                             | 开销比较大，且在高并发模式下影响效率                         |
| 基于回调的非阻塞/异步IO | 高并发服务中使用多线程模式会很快耗尽服务器的内存和CPU资源，这种模式通过事件驱动的方式使用异步IO使服务器持续运转 | 尽可能少用线程，降低开销                                     | 比普通的多线程复杂，因为它做了流程分割，对于问题本身的反应不够自然 |
| 协程                    | 本质上是一种用户态线程，不需要操作系统来进行抢占式调度，且在真正的实现中寄存于线程中 | 系统开销极小，可以有效提高线程的任务并发性而避免多线程的缺点 | 需要语言层面的支持，原生支持协程的语言比较少                 |

## 协程

> 与传统的系统级线程和进程相比，协程最大优势在于其”轻量级“，可以轻松创建上百万个而不会导致系统资源枯竭，而线程和进程通常最多也不能超过1万个。

Golang在语言层面就支持协程goroutine，标准库提供的所有系统调用操作（包括所有同步IO操作）都会出让CPU给其他goroutine，从而让协程的切换管理不依赖于系统的线程和进程，也不依赖于CPU的核心数量。

#### 1. 使用协程

```c++
// 开启协程执行foo函数
go foo()
```

在函数前加上`go`关键字就会启动一个goroutine并发执行这个函数，当被调用的函数返回时这个goroutine也就自动结束了。

#### 2. 协程注意事项

* 如果协程执行的函数有返回值，那么这个返回值会被丢弃
* `main()`函数返回时，程序会直接退出而不等待其他goroutine结束

#### 3. 出让时间片

我们可以使用runtime包中的Gosched()函数控制当前goroutine主动出让时间片给其他goroutine。举个例子，在单核时我们执行如下代码时goroutine还未开始执行主函数就返回了，因此输出的全是Dog。

```go
package main

import (
	"fmt"
	"runtime"
)

func print20times(str string) {
	for i := 0; i < 20; i++ {
		fmt.Println(str)
	}
}

func main() {
	// 设置单核心执行程序
	runtime.GOMAXPROCS(1)

	go print20times("Cat")
	print20times("Dog")
}

// 输出:
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
Dog
```

我们可以使用Gosched()函数让主线程主动出让时间片，此时Dog与Cat就会交错输出：

```go
package main

import (
	"fmt"
	"runtime"
)

func print20times(str string) {
	for i := 0; i < 20; i++ {
		runtime.Gosched()
		fmt.Println(str)
	}
}

func main() {
	// 设置单核心执行程序
	runtime.GOMAXPROCS(1)

	go print20times("Cat")
	print20times("Dog")
}

// 输出:
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
Cat
Dog
```



## Reference

[1] https://www.cnblogs.com/coder-programming/p/10595804.html

[2] https://www.cnblogs.com/sdyxlyb22/p/11604606.html

[3] https://www.jianshu.com/p/4eb593b3a38a