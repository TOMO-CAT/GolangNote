# for循环语句

> Go中的循环语句只支持`for`关键字，而不支持`while`和`do-while`结构。

## for循环写法

#### 1. 基本用法

```go
for i := 0; i < 10; i++ {
    fmt.Println(i)
}

// 输出:
0
1
2
3
4
5
6
7
8
9
```

#### 2. 接单个条件表达式

```go
i := 0
for i < 5 {
    fmt.Println(i)
    i++
}

// 输出
0
1
2
3
4
```

#### 3. 无限循环

```go
i := 0
for {
    fmt.Println(i)
    i++
    if i > 5 {
        break  // 如果不加上退出循环的条件, 会导致死循环
    }
}

// 输出
0
1
2
3
4
5
```

#### 4. for-range语句

> 遍历一个可迭代对象时，可以通过`for-range`来实现，常见的可迭代对象包括数组、切片和字符串等。

```go
intArr := []int{10, 100, 1000}
// range返回两个值: 索引和元素
for index, item := range intArr {
    fmt.Printf("index:%d, item:%d\n", index, item)
}

// 输出
index:0, item:10
index:1, item:100
index:2, item:1000
```

也可以使用一个变量来接收`range`的返回值，这时候接收到的是索引

```go
intArr := []int{10, 100, 1000}
for index := range intArr {
    fmt.Printf("index:%d, item:%d\n", index, intArr[index])
}

// 输出
index:0, item:10
index:1, item:100
index:2, item:1000
```

## 循环控制

Go语言中的`for`循环同样支持`continue`和`break`来控制循环，而且可以根据标签来选择中断哪一个循环：

```go
JLoop:
for j := 0; j < 5; j++ {
    for i:= 0; i < 10; i++ {
        if i > 5 {
            break JLoop  // 中断的是JLoop标签处的外层循环
        }
        fmt.Println(i)
    }
}

// 输出
0
1
2
3
4
5
```

## 不支持逗号分隔的多赋值语句

Go语言的for循环和C++一样都支持在循环条件中定义和初始化变量，唯一的区别是，Go语言不支持以逗号为分隔符的多个赋值语句，必须使用“平行赋值”的方式来初始化多个变量：

```go
a := []int{1, 2, 3, 4, 5, 6}
fmt.Printf("before: %+v\n", a)
for i, j := 0, len(a) - 1; i < j; i, j = i + 1, j - 1 {
    a[i], a[j] = a[j], a[i]
}
fmt.Printf("after: %+v\n", a)

// 输出
before: [1 2 3 4 5 6]
after: [6 5 4 3 2 1]
```



