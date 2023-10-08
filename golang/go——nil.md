# go nil

## 默认值 nil
在go语言中  
- bool 类型的默认值是 false
- 数值类型的默认值是 0
- 字符串类型的默认值是 ""

除此以外的类型默认值都是 nil， 包括
- 指针
- slice
- map
- function
- channel
- interface

## nil 是不能比较的
- 不同类型的nil无法比较
- 相同类型的nil也无法比较，特例nil接口可以比较

```go
func main() {
	var a []int
	var b []int
	fmt.Println(a)        // []
	fmt.Println(b)        // []
	fmt.Printf("%p\n", a) // 0x0
	fmt.Printf("%p\n", b) // 0x0
	fmt.Println(a == nil) // true
	fmt.Println(b == nil) // true
	//fmt.Println(a == b)   // invalid operation: a == b (slice can only be compared to nil)
	
	var c interface{}
	var d interface{}
	fmt.Println(c == d) //true
}
```

## 对值为nil的 slice、map、array、channel指针进行range操作是合法的
- 对slice和map的循环次数是0
- 对array的循环次数取决于数组的长度
- 对channel的range操作会永久阻塞在当前goroutine

```go
func main() {
	for range []int(nil) { //循环次数将是0
		fmt.Println("Hello")
	}

	for range map[string]string(nil) { //循环次数将是0
		fmt.Println("world")
	}

	for i := range (*[5]int)(nil) {
		fmt.Println(i) // 0 1 2 3 4
	}

	for range chan bool(nil) { // block here
		fmt.Println("Bye") //fatal error: all goroutines are asleep - deadlock!
	}
}
```