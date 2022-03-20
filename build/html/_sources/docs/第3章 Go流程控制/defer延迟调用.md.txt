# defer延迟调用语句

## 用法

#### 1. 直接接函数

```go
func myFunc(srcFilePath string) {
	srcFile, err := os.Open(srcFilePath)
	if err != nil {
		return
	}
	defer srcFile.Close()  // 保证srcFile被正常关闭
}
```

#### 2. 接匿名函数

```go
defer func() {
    // 需要在return后处理的操作
} ()
```

## 即时求值的变量快照

`defer`的定位是延迟调用语句，此时传递给函数内的变量，不应该受到后续程序的影响：

```go
import "fmt"

// 即使我们对myStr重新赋值, 在return后调用defer函数仍然使用未被赋值的变量值
func main() {
    myStr := "defer TOMO"
    defer fmt.Println(myStr)  // 输出: defer TOMO

    myStr = "CAT"
    fmt.Println(myStr) // 输出: CAT
}

// 输出
CAT
defer TOMO
```

## 多个defer反序调用

一个函数中可以存在多个`defer`语句，这些`defer`语句的调用遵循先进后出的原则，即最后一个`defer`语句最先被执行。

```go
func main() {
	defer fmt.Println("TOMO")
	defer fmt.Println("CAT")
	defer fmt.Println("TOMO-CAT")
	return
}

// 输出
TOMO-CAT
CAT
TOMO
```

## defer在return后再执行

```go
var MyStr = "TOMO-CAT"

func testFunc() string {
	defer func() {
		MyStr = "CAT"
	}()
	fmt.Printf("testFunc::MyStr:[%s]\n", MyStr)
	return MyStr
}

func main() {
	newStr := testFunc()  // newStr接受到的是TOMO-CAT, 此时还未被defer函数修改为CAT
	fmt.Printf("main::MyStr:[%s]\n", MyStr)
	fmt.Printf("main::newStr:[%s]\n", newStr)
	return
}

// 输出
testFunc::MyStr:[TOMO-CAT]
main::MyStr:[CAT]
main::newStr:[TOMO-CAT]
```

## panic后的defer不可用

如果panic的位置在defer函数之后，那么defer函数可以捕获panic：

```go
package main

import "fmt"

func main() {
	defer func() {
		if err := recover(); err != nil {
			fmt.Printf("panic, err:%v\n", err)
		}
	}()
	
	var pi *int = nil
	fmt.Println(*pi)
	
	return
}

// 输出:
panic, err:runtime error: invalid memory address or nil pointer dereference
```

如果panic的位置在defer函数之前，那么该panic无法被defer函数捕获：

```go
package main

import "fmt"

func main() {
	var pi *int = nil
	fmt.Println(*pi)
	
	defer func() {
		if err := recover(); err != nil {
			fmt.Printf("panic, err:%v\n", err)
		}
	}()
	
	return
}

// 输出:
panic: runtime error: invalid memory address or nil pointer dereference
[signal SIGSEGV: segmentation violation code=0x1 addr=0x0 pc=0x48eac1]

goroutine 1 [running]:
main.main()
	/box/main.go:7 +0x31

Exited with error status 2
```



