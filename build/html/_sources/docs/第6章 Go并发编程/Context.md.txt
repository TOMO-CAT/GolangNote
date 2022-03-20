# Context

## 简介

Go1.7之后正式将golang.org/x/net/context包中的Context纳入标准库，它主要用于保存goroutine的上下文（运行状态、环境和现场等信息），几乎成为了并发控制和超时控制的标准做法。

举个rpc服务端开发的例子：对于同一个http/thrift请求可能会开启多个同步/异步的goroutine对请求进行处理，这些基于同一个请求的goroutine一方面需要共享一些信息，另一方面当request超时时也应该被提前结束，这种情况下正是Conetxt大展拳脚的时候。

## 定义

Context也被称为上下文，它是一个接口类型，定义如下：

```go
// A Context carries a deadline, a cancellation signal, and other values across
// API boundaries.
//
// Context's methods may be called by multiple goroutines simultaneously.
type Context interface {
	// Deadline returns the time when work done on behalf of this context
	// should be canceled. Deadline returns ok==false when no deadline is
	// set. Successive calls to Deadline return the same results.
	Deadline() (deadline time.Time, ok bool)

	// Done returns a channel that's closed when work done on behalf of this
	// context should be canceled. Done may return nil if this context can
	// never be canceled. Successive calls to Done return the same value.
	// The close of the Done channel may happen asynchronously,
	// after the cancel function returns.
	//
	// WithCancel arranges for Done to be closed when cancel is called;
	// WithDeadline arranges for Done to be closed when the deadline
	// expires; WithTimeout arranges for Done to be closed when the timeout
	// elapses.
	//
	// Done is provided for use in select statements:
	//
	//  // Stream generates values with DoSomething and sends them to out
	//  // until DoSomething returns an error or ctx.Done is closed.
	//  func Stream(ctx context.Context, out chan<- Value) error {
	//  	for {
	//  		v, err := DoSomething(ctx)
	//  		if err != nil {
	//  			return err
	//  		}
	//  		select {
	//  		case <-ctx.Done():
	//  			return ctx.Err()
	//  		case out <- v:
	//  		}
	//  	}
	//  }
	//
	// See https://blog.golang.org/pipelines for more examples of how to use
	// a Done channel for cancellation.
	Done() <-chan struct{}

	// If Done is not yet closed, Err returns nil.
	// If Done is closed, Err returns a non-nil error explaining why:
	// Canceled if the context was canceled
	// or DeadlineExceeded if the context's deadline passed.
	// After Err returns a non-nil error, successive calls to Err return the same error.
	Err() error

	// Value returns the value associated with this context for key, or nil
	// if no value is associated with key. Successive calls to Value with
	// the same key returns the same result.
	//
	// Use context values only for request-scoped data that transits
	// processes and API boundaries, not for passing optional parameters to
	// functions.
	//
	// A key identifies a specific value in a Context. Functions that wish
	// to store values in Context typically allocate a key in a global
	// variable then use that key as the argument to context.WithValue and
	// Context.Value. A key can be any type that supports equality;
	// packages should define keys as an unexported type to avoid
	// collisions.
	//
	// Packages that define a Context key should provide type-safe accessors
	// for the values stored using that key:
	//
	// 	// Package user defines a User type that's stored in Contexts.
	// 	package user
	//
	// 	import "context"
	//
	// 	// User is the type of value stored in the Contexts.
	// 	type User struct {...}
	//
	// 	// key is an unexported type for keys defined in this package.
	// 	// This prevents collisions with keys defined in other packages.
	// 	type key int
	//
	// 	// userKey is the key for user.User values in Contexts. It is
	// 	// unexported; clients use user.NewContext and user.FromContext
	// 	// instead of using this key directly.
	// 	var userKey key
	//
	// 	// NewContext returns a new Context that carries value u.
	// 	func NewContext(ctx context.Context, u *User) context.Context {
	// 		return context.WithValue(ctx, userKey, u)
	// 	}
	//
	// 	// FromContext returns the User value stored in ctx, if any.
	// 	func FromContext(ctx context.Context) (*User, bool) {
	// 		u, ok := ctx.Value(userKey).(*User)
	// 		return u, ok
	// 	}
	Value(key interface{}) interface{}
}
```

* `Deadline()`：返回Context被取消的时间，如果没有设置deadline则返回的ok为false
* `Done()`：返回一个channel，此channel在Context被取消时关闭，此channel为nil时意味着这个Context永远不会被取消
* `Err()`：仅有在`Done()`方法返回的channel被关闭时才会返回非空值，返回Canceled错误表示Context已经被取消，返回DeadlineExceeded错误表示该Context已经达到了deadline
* `Value()`：返回此Context中某个key对应的值，不存在该key时返回nil，需要注意的是该方法只应该用在程序和API中传递和请求相关的元数据，而不应该被用于传递可选的参数

## 使用Context的原因

前面我们提到可以使用select和channel对创建的goroutine进行超时限制：

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

func main() {
	wg := sync.WaitGroup{}
	wg.Add(1)

	// 使用channel进行超时限制, 超时时间为10秒
	timeout := make(chan bool, 1)
	go func() {
		time.Sleep(10 * time.Second)
		timeout <- true
	}()

	// 异步出去的goroutine, 超时后自动停止
	go func() {
		defer wg.Done()
		ticker := time.NewTicker(1 * time.Second)
		for _ = range ticker.C {
			select {
			case <- timeout:
				fmt.Println("timeout and quit")
				return
			default:
				fmt.Println("do something")
			}
		}
	}()

	wg.Wait()
}

// 输出:
do something
do something
do something
do something
do something
do something
do something
do something
do something
timeout and quit
```

使用context后我们不仅可以更加简洁地进行goroutine的超时控制，还可以在任意时间点或者任意事件触发时取消context，还可以实现上下文信息在goroutine之间传递。

> 使用WithCancel函数可以返回一个context对应的取消函数，当这个函数被调用时就会取消context。此外我们还可以使用WithDeadline和WithTimeout函数，前者用于定时取消而后者用于超时取消。



#### 1. 取消单个goroutine

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancelFunc := context.WithCancel(context.Background())

	// goroutine: 每秒循环处理事务
	go func() {
		for {
			select {
			case <-ctx.Done():
				fmt.Println("done and quit")
				return
			default:
				fmt.Println("do something...")
				time.Sleep(1 * time.Second)
			}
		}
	}()

	time.Sleep(10 * time.Second)
	// 控制goroutine结束
	cancelFunc()
	time.Sleep(3 * time.Second)
}

// 输出:
do something...
do something...
do something...
do something...
do something...
do something...
do something...
do something...
do something...
do something...
do something...
done and quit

Process finished with the exit code 0
```

#### 2. 取消多个goroutine

```go
package main

import (
	"context"
	"fmt"
	"time"
)

// 工作函数: 每秒循环处理事务
func worker(ctx context.Context, index int) {
	for {
		select {
		case <-ctx.Done():
			fmt.Printf("[worker %d]done and quit\n", index)
			return
		default:
			fmt.Printf("[worker %d]do something...\n", index)
			time.Sleep(1 * time.Second)
		}
	}
}

func main() {
	ctx, cancelFunc := context.WithCancel(context.Background())

	// 启动三个goroutine
	go worker(ctx, 0)
	go worker(ctx, 1)
	go worker(ctx, 2)

	time.Sleep(5 * time.Second)
	// 控制goroutine结束
	cancelFunc()
	time.Sleep(1 * time.Second)
}

// 输出:
[worker 2]do something...
[worker 1]do something...
[worker 0]do something...
[worker 0]do something...
[worker 1]do something...
[worker 2]do something...
[worker 0]do something...
[worker 1]do something...
[worker 2]do something...
[worker 1]do something...
[worker 2]do something...
[worker 0]do something...
[worker 0]do something...
[worker 2]do something...
[worker 1]do something...
[worker 2]done and quit
[worker 1]done and quit
[worker 0]done and quit
```

#### 3. 传递上下文

context当然附带有传递上下文信息的功能，可以使用WithValue函数增加键值对：

```go
package main

import (
	"context"
	"fmt"
	"time"
)

func main() {
	ctx, cancelFunc := context.WithCancel(context.Background())

	// 设置对应的上下文字段
	ctxValue := context.WithValue(ctx, "name", "tomocat")

	go func() {
		for {
			select {
			case <-ctxValue.Done():
				fmt.Printf("cancel context, name:%v\n", ctxValue.Value("name"))
				return
			default:
				fmt.Println("do something...")
				time.Sleep(1 * time.Second)
			}
		}
	}()

	time.Sleep(5 * time.Second)
	cancelFunc()
	time.Sleep(1 * time.Second)
}

// 输出:
do something...
do something...
do something...
do something...
do something...
cancel context, name:tomocat
```

## 官方提供的四类Context实现

前面提到Context本质上是一个接口，只要实现了它定义的四个方法就可以认为实现了Context接口。四类Context如下所示：

| 实现      | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| emptyCtx  | 空Context，实现的函数也是返回nil，仅仅是实现了Context接口而已 |
| cancelCtx | 继承自Context，同时实现了cancel方法                          |
| timerCtx  | 继承自cancelCtx，增加了timeout机制                           |
| valueCtx  | 存储键值对数据                                               |

## Reference

[1] https://zhuanlan.zhihu.com/p/110085652

[2] http://c.biancheng.net/view/5714.html

[3] https://zhuanlan.zhihu.com/p/110085652

[4] https://blog.csdn.net/u011957758/article/details/82948750

[5] https://www.jianshu.com/p/dcbd87eb1a3f

[6] https://developer.51cto.com/art/202104/660097.htm