## go知识点总结

### go的垃圾回收机制

> [一文搞懂go gc垃圾回收原理](https://juejin.cn/post/7111515970669117447)

1. 永远不要使用一个指针指向一个**接口类型**，因为它已经是一个指针

```go
// 正确
func f(x interface()) {
}

// 错
func f(x *interface()) {
}
```

1. 当且仅当动态值和动态类型都为 nil 时，**接口类型**值才为 nil
2. 对于类似 X = Y的赋值操作，必须知道 X 的地址，才能够将 Y 的值赋给 X，但 go 中的 map 的 value 本身是不可寻址的，因此value不可修改，只能整体替换
```go
type Math struct {
    x, y int
}

var m = map[string]Math{
    "foo": Math{2, 3},
}

func main() {
    // 错误
    // m["foo"].x = 4
    m["foo"] = Math{4,3}
    fmt.Println(m["foo"].x)
}
```
1. go 中不同类型是不能比较的，而数组长度是数组类型的一部分，所以 […]int{1} 和 [2]int{1} 是两种不同的类型，不能比较；
2. **切片是不能比较**的
3. for range 使用短变量声明(:=)的形式迭代变量，需要注意的是，变量 i、v 在每次循环体中都会被重用，而不是重新声明 >> [例子](http://mian.topgoer.com/%E7%AC%AC%E4%BA%8C%E5%8D%81%E4%B9%9D%E5%A4%A9/)