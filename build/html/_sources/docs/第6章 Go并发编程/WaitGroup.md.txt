# WaitGroup

## 标记完成

main函数返回时，程序会直接退出而不等待其他goroutine结束，因此在下面这个程序看不到goroutine的输出：

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
    // 设置单核心执行程序, 避免多核心带来的影响(多核下子协程更容易获得执行机会)
	runtime.GOMAXPROCS(1)

	for i := 0; i < 10; i++ {
		go func(index int) {
			fmt.Println(index)
		}(i)
	}

	fmt.Println("done")
}

// 输出:
done
```

#### 1. 使用sleep函数

当我们想要获取一个异步goroutine的运算结果时，需要一个机制来标记goroutine是否完成。golang初学者为了保证主线程退出前goroutine可以执行结束，往往使用time包的sleep函数保证goroutine执行结束：

```go
package main

import (
	"fmt"
	"runtime"
	"time"
)

func main() {
	// 设置单核心执行程序, 避免多核心带来的影响(多核下子协程更容易获得执行机会)
	runtime.GOMAXPROCS(1)

	for i := 0; i < 10; i++ {
		go func(index int) {
			fmt.Println(index)
		}(i)
	}

	time.Sleep(1 * time.Second)
	fmt.Println("done")
}

// 输出:
9
0
1
2
3
4
5
6
7
8
done
```

但在业务开发中这是一种极其不推荐的方法，因为我们无法预知每个goroutine的执行时间，如果sleep的时间过长就会阻塞主协程造成浪费，如果sleep的时间过短goroutine就无法完成。

#### 2. 使用channel实现通信

golang中我们可以使用channel实现协程间通信：

```go
package main

import (
	"fmt"
	"runtime"
)

func main() {
	// 设置单核心执行程序, 避免多核心带来的影响(多核下子协程更容易获得执行机会)
	runtime.GOMAXPROCS(1)

	flag := make(chan bool, 10)

	for i := 0; i < 10; i++ {
		go func(index int) {
			fmt.Println(index)
			flag <- true
		}(i)
	}

	for i := 0; i < 10; i++ {
		<-flag
	}

	fmt.Println("done")
}

// 输出:
9
0
1
2
3
4
5
6
7
8
done
```

#### 3. 使用WaitGroup

可以看到当我们需要wait的协程数较多时使用channel实现通信的方式就比较复杂，代码可阅读性较差。WaitGroup可以更优雅的协程同步：

```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

func main() {
	// 设置单核心执行程序, 避免多核心带来的影响(多核下子协程更容易获得执行机会)
	runtime.GOMAXPROCS(1)

	var wg sync.WaitGroup
	wg.Add(10)
	
	for i := 0; i < 10; i++ {
		go func(index int) {
			defer wg.Done()
			fmt.Println(index)
		}(i)
	}

	wg.Wait()
	fmt.Println("done")
}

// 输出:
9
0
1
2
3
4
5
6
7
8
done
```

## WaitGroup对外接口

WaitGroup内部维护了一个计数器，提供了三个接口：

* Add()：计数器加一
* Done()：计数器减一
* Wait()：阻塞直至计数器清零









