# goto跳转语句

## 简介

`goto`语句的功能是跳转到本函数内的某个标签，例如：

```go
import "fmt"

func main() {
    goto FLAG
    fmt.Println("TOMO")
FLAG:
    fmt.Println("CAT")
}

// 输出
CAT
```

## 建议

如非必要，否则不要使用`goto`语句，容易增加代码的维护成本。

## 注意

`goto`语句和标签之间不能存在变量声明，否则会报编译错误：`goto FLAG jumps over declaration of myString`。

```go
import "fmt"

func main() {	
	goto FLAG
	var myString = "TOMOCAT"
	fmt.Println("TOMO")
FLAG:
	fmt.Println("CAT")
	fmt.Println(myString)
}
```

