# map

> Go中map是一堆键值对的未排序集合。

## 基础语法

#### 1. 变量声明

```go
var myMap map[string]string
```

#### 2. 创建

```go
// 使用make()创建map
myMap = make(map[string]string)

// 使用make()创建map并指定map的初始存储能力
myMap = make(map[string]string, 100)

// 创建并初始化map
myMap = map[string]string {
    "key1": "value1",
    "key2": "value2",
}
```

#### 3. 元素赋值

```go
// 如果key不存在, 则相当于新增元素
myMap["key3"] = "value3"
```

#### 4. 元素删除

Go提供一个内置函数`delete()`用于删除容器内的元素，但是需要注意：

* 如果传入的key不存在，那么这个调用不会有任何作用也不会报错
* 如果传入的map为nil，那么程序会抛出异常`panic`

```go
// 调用delete前必须保证map不为nil
delete(myMap, "key1")
```

#### 5. 元素查找

```go
value, ok := myMap["key1"]
if ok {
    // 成功查找到元素
}
```



