# 结构体中的tag

## 例子

结构体的字段可以额外增加一个或者多个标签，比如json序列化时用到的标签或者数据库字段名的标签：

```go
type MyBook struct {
	Name   string  `json:"name"`
	Author string  `json:"author"`
	Price  float64 `json:"price"`
}

func main() {
	book := MyBook{
		Name:   "CAT",
		Author: "TOMO",
		Price:  100,
	}

	if bookStr, err := json.Marshal(book); err == nil {
		fmt.Println(string(bookStr))
	}

	return
}

// 输出
{"name":"CAT","author":"TOMO","price":100}
```

包含匿名结构体的tag：

```go
type Request struct {
	Meta struct {
		Caller      string `json:"caller"`
		TraceID     string `json:"traceId"`
		HintCode    int64  `json:"hintCode"`
		HintContent string `json:"hintContent"`
	} `json:"meta"`
	Request struct {
		Caller string `json:"caller"`
	} `json:"request"`
	Common struct {
		Lang            string `json:"lang"`
		LocationCountry string `json:"location_country"`
		LocationCityID  int64  `json:"location_cityid"`
	} `json:"common"`
}
```

## 通过反射获取tag

```go
type MyBook struct {
	Name   string  `json:"name" sql:"test"`
	Author string  `json:"author"`
	Price  float64 `json:"price"`
}

func main() {
	book := MyBook{
		Name:   "CAT",
		Author: "TOMO",
		Price:  100,
	}

	// 法一: 通过字段名获取字段
	if fieldName, ok := reflect.TypeOf(book).FieldByName("Name"); ok {
		tag := fieldName.Tag
		// tag.Get方法是对tag.Lookup方法的封装, 获取不到tag时返回空字符串
		jsonTagValue := tag.Get("json")
		fmt.Println("jsonTagValue: ", jsonTagValue)
		if sqlTagValue, ok := tag.Lookup("sql"); ok {
			fmt.Println("sqlTagValue: ", sqlTagValue)
		}
	}

	// 法二: 通过字段顺序获取字段
	{
		fieldName := reflect.ValueOf(book).Type().Field(0)
		tag := fieldName.Tag
		jsonTagValue := tag.Get("json")
		fmt.Println("jsonTagValue: ", jsonTagValue)
	}

	// 法三: 通过指针获取指针指向对象的字段
	{
		fieldName := reflect.ValueOf(&book).Elem().Type().Field(0)
		tag := fieldName.Tag
		jsonTagValue := tag.Get("json")
		fmt.Println("jsonTagValue: ", jsonTagValue)
	}

	return
}
```

