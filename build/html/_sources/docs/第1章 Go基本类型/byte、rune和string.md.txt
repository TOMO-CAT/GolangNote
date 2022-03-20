# Golang byte、rune和string

## string

#### 1. 字符串操作

| 操作        | 含义     | 例子                       |
| ----------- | -------- | -------------------------- |
| str1 + str2 | 连接     | "TOMO" + "CAT" = "TOMOCAT" |
| len(str)    | 获取长度 | len("TOMOCAT") = 7         |
| str[i]      | 下标访问 | "TOMOCAT"[2] = 'M'         |

#### 2. string特性

* 字符串的内容不能在初始化后被修改

```go
str := "TOMOCAT"
str[1] = 'M' // 编译错误
```

* string本质上是byte数组

```go
str := []byte{84, 79, 77, 79}
fmt.Printf("str:[%s]\n", str)

// 输出:
// str:[TOMO]
```

* string使用UTF-8编码

Go编译器支持UTF-8的源代码文件格式，这意味着源代码中的字符串可以包含非ANSI的字符。其中每个英文字母占用1个字节，每个中文汉字占用3个字节。

* 双引号表示法与反引号表示法

Go语言支持string两种表示方法，可以使用双引号或者反引号包裹字符串，前者被称为“解释型表示法”，后者被称为“原生型表示法”（所见即所得）。

```go
str1 := "\\n"
str2 := `\n`
fmt.Printf("str1:[%s]\n", str1)
fmt.Printf("str2:[%s]\n", str2)
// 输出:
// str1:[\n]
// str2:[\n]

/* 可以通过%q格式控制符打印解释型字符 */
fmt.Printf("str3:[%q]\n", `\t\n\r`)
// 输出:
// str3:["\\t\\n\\r"]
```

#### 3. 遍历字符串

```go
str := "TOMOCAT"

/* 以Unicode字符遍历 */
for i, ch := range str {
    fmt.Println(i, ch)  // ch类型为rune
}

/* 以字节数组遍历 */
for i := 0; i < len(str); i++ {
    fmt.Println(i, str[i])
}
```

## byte与rune

byte（实际上是unit8的别名），表示范围0~255，代表ASCII表中的一个字符。rune占4个字节，表示一个Unicode字符。无论是使用byte还是rune定义一个字符时都应该使用单引号：

```go
var ch1 rune = 'T'
var ch2 byte = 'O'
```

> byte本质上和unit8没有区别，rune和uint32也没有区别，仅仅是通过类型别名让它们更直观地表示一个字符。

#### 1. 打印单个字符

> 单个字符的格式控制为`%c`

```go
/* byte本质上等于uint8 */
var ch1 byte = 65
var ch2 uint8 = 66
fmt.Printf("ch1:[%c], ch2:[%c]\n", ch1, ch2)

// 输出:
// ch1:[A], ch2:[B]
```

#### 2. rune和byte的大小

> rune占4个字节而byte占1个字节

```go
/* rune占4个字节, byte占1个字节 */
var chByte byte = 'A'
var chRune rune = 'B'
fmt.Printf("chByte size:[%d]\n", unsafe.Sizeof(chByte))
fmt.Printf("chRune size:[%d]\n", unsafe.Sizeof(chRune))

// 输出:
chByte size:[1]
chRune size:[4]
```



