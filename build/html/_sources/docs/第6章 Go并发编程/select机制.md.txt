# select机制

## 简介

select在Unix时代就已经被设计出来用于监控一系列的文件句柄，一旦其中一个文件句柄发生了IO动作那么该select调用就会被返回，后来该机制也被用于实现高并发的Socket服务器程序。Go在语言层面支持select关键字，用于处理异步IO问题。

## select使用

和switch不同，select中每条case语句里都必须是一个channel操作，如果所有的case分支都未能成功执行，那么直接进入default语句。

```go
package main

import "fmt"

func main() {
	ch1 := make(chan int, 1)
	ch1 <- 10
	select {
	case res := <-ch1:
		fmt.Println("receive from ch1: ", res)
	default:
		fmt.Println("can not receive from ch1")
	}
}

// 输出:
receive from ch1:  10
```

需要注意的是select语句中的case分支是随机执行的，比如下面这面例子实现了随机输出0和1的功能：

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 1)
	for i := 0; i < 10; i++ {
		select {
		case ch <- 1:
		case ch <- 0:
		}
		fmt.Println("receive from ch: ", <-ch)
	}
}

// 输出:
receive from ch:  0
receive from ch:  0
receive from ch:  0
receive from ch:  1
receive from ch:  0
receive from ch:  0
receive from ch:  1
receive from ch:  0
receive from ch:  1
receive from ch:  0
```

## select解决超时问题

golang并发编程中向已满的channel写入数据或者从空channel中读取数据会导致整个goroutine阻塞住，我们需要自己处理此类超时问题。

一种简单的方法是使用default分支处理阻塞问题，当阻塞时会自动进入default分支：

```go
package main

import "fmt"

func main() {
	ch := make(chan int, 5)
	for i := 0; i < 10; i++ {
		select {
		case ch <- i:
			fmt.Println("send to ch successfully")
		default:
			fmt.Println("ch is full")
		}
	}
}

// 输出:
send to ch successfully
send to ch successfully
send to ch successfully
send to ch successfully
send to ch successfully
ch is full
ch is full
ch is full
ch is full
ch is full
```

使用default分支会导致阻塞时直接退出，我们也可以通过另一个channel来实现超时退出机制：

```go
package main

import (
	"fmt"
	"time"
)

func main() {
	ch := make(chan int, 1)

	timeout := make(chan bool, 1)
	go func() {
		time.Sleep(1 * time.Second)
		timeout <- true
	}()

	select {
	case <-ch:
		fmt.Println("receive from ch successfully")
	case <-timeout:
		fmt.Println("timeout")
	}
}

// 输出:
timeout
```

