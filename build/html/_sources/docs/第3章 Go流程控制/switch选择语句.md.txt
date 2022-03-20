# switch选择语句

## 写法

```go
var i int
switch i {
case 0:
    fmt.Println("0")
case 1:
    fmt.Println("1")
case 2:
    fallthrough
case 3:
    fmt.Println("3")
case 4, 5, 6:
    fmt.Println("4, 5, 6")
default:
    fmt.Println("others")
}
```

* i为0：输出"0"
* i为1：输出"1"
* i为2和3：输出"3"
* i为4、5和6：输出"4,5,6"
* i为其他值：输出"others"

## 注意

#### 1. fallthrough关键字

>  和C++不同的是，如果case条件后没有接任何语句块，那么相当于该case满足时不做任何操作。如果要指明继续执行紧跟着的下一个case的语句块，那么需要添加`fallthrough`关键字。

正常情况下，只要有一个case满足条件，那么就会直接退出`switch`结构体。如果我们使用关键字`fallthrough`就会直接执行下一个`case`语句，而且不需要判断条件。但是`fallthrough`只会穿透一层，执行完下一个case语句后就退出`switch`结构体。

```go
// 使用fallthrough: 输出"default"
i := 1
switch i {
case 0:
    fmt.Println("0")
case 1:
	fallthrough
default:
    fmt.Println("default")
}

// 不使用: 无任何输出
i := 1
switch i {
case 0:
    fmt.Println("0")
case 1:
default:
    fmt.Println("default")
}
```

#### 2. 单个case后可跟多个条件

一个case后可跟多个条件，每个条件之间是“或”的关系：

```go
// i为4或5或6时输出都是"4, 5, 6"
var i int
switch i {
case 0:
    fmt.Println("0")
case 1:
    fmt.Println("1")
case 2:
    fallthrough
case 3:
    fmt.Println("3")
case 4, 5, 6:
    fmt.Println("4, 5, 6")
default:
    fmt.Println("others")
}
```

#### 3. switch后可不接表达式

> `switch`后可以不跟表达式，这样子的话整个`switch`结构与多个`if...else...`结构类似。

```go
score := 10
switch {
case score >= 60 && score <= 100:
    fmt.Println("pass")
case score < 60 && score >= 0:
    fmt.Println("fail")
default:
    fmt.Println("invalid score")
}
```

#### 4. 不需要break来明确表示退出一个case

和C++不同的是，Go语言下的switch无论是否在case语句块中使用break，都会在执行完该case语句块后跳出switch结构体。

#### 5. case条件常量不可重复

> 当`case`后接的是常量时，该常量只能出现一次。

下面的错误案例在编译时会报错：`duplicate case “male” in switch`。

```go
gender := "male"

switch gender {
case "male":
    fmt.Println("男性")
case "male":  // 与上面重复
    fmt.Println("男性")
case "female":
    fmt.Println("女性")
}
```

