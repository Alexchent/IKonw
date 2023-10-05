# go 零切片，空切片，nil切片

## 零切片
我们把底层数组的值都是0或nil的切片成为零切片，**使用`make`创建的长度容量都不为0的切片就是0切片**
```go
slice1 := make([]int, 3)   // 0 0 0
slice1 := make([]*int, 3)  // nil nil nil
```

### 空切片
空切片的长度和容量都为0，但是和nil比较的结果为false，因为所有切片的指针都指向同一个地址；使用**字面量**和**make**创建的切片就是空切片
```go
slice1 := []int{}
slice2 := make([]int, 0)
```

### nil切片
nil切片（nil slice）的长度和容量都为0，并且和nil比较的结果为true，采用**直接创建切片**的方式、**new创建切片**的方式都可以创建nil切片：
```go
var slice1 []int
slice2 := new([]int)
```

> 常用于返回slice的函数，当函数异常，保障依然有nil返回值
