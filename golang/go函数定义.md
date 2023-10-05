## go 函数

go 函数定义格式如下：
```
func function_name( [parameter list] ) [return_types] {
   函数体
}
```

- func：函数由 func 开始声明
- function_name：函数名称，参数列表和返回值类型构成了函数签名。
- parameter list：参数列表，参数就像一个占位符，当函数被调用时，你可以将值传递给参数，这个值被称为实际参数。参数列表指定的是参数类型、顺序、及参数个数。参数是可选的，也就是说函数也可以不包含参数。
- return_types：返回类型，函数返回一列值。return_types 是该列值的数据类型。有些功能不需要返回值，这种情况下 return_types 不是必须的。
- 函数体：函数定义的代码集合。

### 无返回值
```
func mind(id int, name, phone string) {
    fmt.Println(id, name, phone)
}
```

### 变参
 可以传 **0** 或 **多个** 参数
```
func Greeting(prefix string, who ...string) {
	fmt.Println(who)
	for k, name := range who {
		fmt.Println(k, prefix, name)
	}
}

// 调用
func main() {
    Greeting("hello", "哈迪斯"， "海格")
}
```

### 多返回值
```
func echo() (string, string, string) {
	return "golang", "php" , "java"
}

func echo2() (id int, b,c string) {
	return 1, "php" , "世界上最好的语言"
}
```