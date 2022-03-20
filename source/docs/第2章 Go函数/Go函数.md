# Go函数

## 函数定义

```go
func Sum(num1 int, num2 int) (res int) {
	return num1 + num2
}
```

## 不定参数

函数接收的参数可以为不定参数（即传入的参数个数不限）。

```go
func PrintArgs(args ...interface{}) {  // 任意类型的不定参数
	for _, arg := range args {
		fmt.Println(arg)
	}
}

func main() {
	PrintArgs(10, "TOMOCAT", 2.33)
	return
}

// 输出
10
TOMOCAT
2.33
```

## 多返回值

与其他语言不同，Go的函数支持多返回值。如果调用方只想获取部分返回值，可以使用`_`跳过某个返回值。

## 匿名函数

> 未指定函数名的函数被称为匿名函数。

匿名函数可以赋值给一个变量或者直接执行。

#### 1. 赋值给函数变量

```go
func main() {
	sumFun := func(num1, num2 int) int {
		return num1 + num2
	}
	sum := sumFun(10, 20)
	fmt.Println(sum)

	return
}

// 输出
30
```

#### 2. 直接执行

```go
func main() {
	func(name string) {
		fmt.Println("Hello", name)
	}("TOMOCAT")

	return
}

// 输出
Hello TOMOCAT
```

#### 3. 作为函数参数

可以定义一个接收匿名函数参数的函数，实现回调的效果。

```go
/*
求和并调用callback函数对结果进行特殊处理
 */
func sumWorker(data []int, callback func(int)) {
	sum := 0
	for _, num := range data {
		sum += num
	}

	callback(sum)
}

func main() {
	// 打印出求和结果
	sumWorker([]int{1, 2, 3, 4}, func(a int) {
		fmt.Println("sum:", a)
	})

	// 判断求和结果是否大于100
	sumWorker([]int{1, 2, 3, 4}, func(a int) {
		if a > 100 {
			fmt.Println("sum > 100")
		} else {
			fmt.Println("sum <= 100")
		}
	})
}

// 输出
sum: 10
sum <= 100
```

## 闭包

闭包是由函数及其相关引用环境组成的实体，可以理解为一个函数“捕获”了和它处于同一作用域的其他变量。

> Golang中所有的匿名函数都是闭包。

#### 1. 理解“捕获”的概念

“捕获”的本质就是引用传递而非值传递。

```go
func main() {
	i := 0

	// 闭包: i是引用传递
	defer func() {
		fmt.Println("defer closure i:", i)
	}()

	// 非闭包: i是值传递
	defer fmt.Println("defer i:", i)

	// 修改i的值
	i = 100
    
	return
}

// 输出
defer i: 0
defer closure i: 100
```

#### 2. 匿名函数与自由变量组成闭包

下面这个例子中实现了Go中常见的闭包场景，我们通过`Adder()`返回一个匿名函数，这个匿名函数和自由变量`x`组成闭包，只要匿名函数的实例`closure`没有消亡，那么`x`都是引用传递。

```go
/*
返回匿名函数的函数
1) x是自由变量
2) 匿名函数和x自由变量共同组成闭包
 */
func Adder(x int) func(int) int{
	return func(y int) int{
		x += y
		fmt.Printf("x addr %p, x value %d\n", &x, x)
		return x
	}
}

func main() {
	fmt.Println("----------------Adder()返回的匿名函数实例1----------------")
	closure := Adder(1)
	closure(100)
	closure(1000)
	closure(10000)

	fmt.Println("----------------Adder()返回的匿名函数实例2----------------")
	closure2 := Adder(10)
	closure2(1)
	closure2(1)
	closure2(1)
	closure2(1)

	return
}

// 输出
----------------Adder()返回的匿名函数实例1----------------
x addr 0xc00002a958, x value 101
x addr 0xc00002a958, x value 1101
x addr 0xc00002a958, x value 11101
----------------Adder()返回的匿名函数实例2----------------
x addr 0xc00002a978, x value 11
x addr 0xc00002a978, x value 12
x addr 0xc00002a978, x value 13
x addr 0xc00002a978, x value 14
```

#### 3. 闭包中使用值传递

由于闭包的存在，Golang中使用匿名函数的时候要特别注意区分清楚引用传递和值传递。根据实际需要，我们在不需要引用传递的地方通过匿名函数参数赋值的方式实现值传递。

```go
func main() {
	fmt.Println("----------------引用传递----------------")
	for i := 0; i < 10; i++ {
		go func() {
			fmt.Println(i)
		}()
	}
	time.Sleep(10 * time.Millisecond)

	fmt.Println("----------------值传递----------------")
	for i := 0; i < 10; i++ {
		go func(x int) {
			fmt.Println(x)
		}(i)
	}
	time.Sleep(10 * time.Millisecond)

	return
}

// 输出
----------------引用传递----------------
1
10
10
10
10
10
10
10
10
10
----------------值传递----------------
4
9
1
6
5
7
2
3
8
0
```

#### 4. 总结

* 匿名函数及其“捕获”的自由变量被称为闭包

* 被闭包捕获的变量称为“自由变量”，在匿名函数实例未消亡时共享同个内存地址
* 同一个匿名函数可以构造多个实例，每个实例内的自由变量地址不同
* 匿名函数内部的局部变量在每次执行匿名函数时地址都是变换的
* 通过匿名函数参数赋值的方式可以实现值传递

## Reference

[1] https://www.cnblogs.com/dcz2015/p/10498343.html