# select语句

## 简介

> 早就Unix时代，`select`机制就已经被引入。通过调用`select()`函数来监控一系列的文件句柄，一旦其中一个文件句柄发生了IO动作，该`select() `调用就会返回。Go直接在语言层面支持`select`关键字，用于处理异步IO问题。

`select`的用法和`switch`语言非常类似，但是每个`case`语句里必须是一个`channel`操作，大致的结构如下：

```go
select {
case <- chan1:
    // 如果chan1成功读到数据, 则进行该case处理语句
case chan2 <- 1:
    // 如果成功向chan2写入数据, 则进行该case处理语句
default:
	// 如果上面都没有成功, 则进入default处理流程    
}
```

## case本身没有先后顺序

> `switch`语句中的case是顺序执行的，和`select`语句不同。

下面这个程序会随机输出0或者1，意味着两个case语句是随机执行的。

```go
func main() {
	ch := make(chan int, 1)
	for j := 0; j < 10; j++ {
		select {
		case ch <- 0:
		case ch <- 1:
		}
		i := <-ch
		fmt.Println("Value received:", i)
	}
	return
}

// 输出
Value received: 0
Value received: 0
Value received: 1
Value received: 0
Value received: 1
Value received: 0
Value received: 1
Value received: 0
Value received: 0
Value received: 0
```

## 避免死锁

`select`语句在执行过程中必须命中其中某一分支，如果没有命中任何一个`case`且没有写`default`语句，那么`select`就会阻塞住直到某个`case`语句被命中。

```go
func main() {
	c1 := make(chan string, 1)
	c2 := make(chan string, 1)

    // 死锁
	select {
	case msg1 := <-c1:
		fmt.Println("c1 received: ", msg1)
	case msg2 := <-c2:
		fmt.Println("c2 received: ", msg2)
	}
	return
}
```

## 超时机制

前面提到当`select`语句没有命中任何一个`case`语句且没有写`default`语句时，那么就会阻塞住。

```go
func main() {
	ch := make(chan int, 1)
	select {
	case i := <-ch:
		fmt.Println("value received:", i)
	case <-time.After(3 * time.Second): // 3秒到期后超时退出
		fmt.Println("timeout, quit")
	}

	return
}

// 输出
timeout, quit
```

