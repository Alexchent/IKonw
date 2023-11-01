# go 细节

## slice和数组的区别
总结:
1. 数组是同一类型元素的固定长度序列，slice的长度则是不固定的。
2. 数组的内存空间在定义时分片，大小是固定的；slice的内存空间是运行时动态分配，大小可变
3. 数组作为函数的参数时，函数操作的是数组的副本，不会影响原始数组；slice作为函数参数时，函数操作的是切片的引用，会影响原始切片

> [Go 语言数组和切片的区别](https://mp.weixin.qq.com/s/esaAmAdmV4w3_qjtAzTr4A)

## slice的扩容机制

**go 1.18**不在以1024作为临界点，而是设置了一个值为**256**的**threshold**，超过这个临界点，不再是每次扩容1/4，而是每次增加（旧容量+3*256)/4

- 当新切片需要的容量大于两倍扩容的容量，则直接按照新切片需要的容量扩容
- 当原容量 < threshold，新slice的容量扩容为原来的2倍
- 当原容量 > threshold，新slice的容量增加（原容量+3*256）/4

另外go还会进行一次内存对齐

>[slice的扩容机制](https://mp.weixin.qq.com/s/VVM8nqs4mMGdFyCNJx16_g)

### map 的赋值操作
对于类似 X = Y的赋值操作，必须知道 X 的地址，才能够将 Y 的值赋给 X，但 go 中的 **map 的 value 本身是不可寻址的**，因此value不可修改，只能整体替换
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

### init() 函数的执行顺序
执行顺序：import –> const –> var –>init()–>main()
1. 一个package甚至同一个源文件可以有多个init函数，它们的执行顺序是不确定的
2. init在main函数前面执行
3. 每个包，先初始化包作用域的常量和变量，再执行init


## new 与 make的区别
1. make只能用于`slice`、`map`、`chan`类型的初始化，new可以分配任意类型的数据
2. new返回的是一个指针，即类型`*Type`；make返回类型本身
3. new分配的内存为空，make分配内存后会初始化

> https://mp.weixin.qq.com/s/NBDkI3roHgNgW1iW4e_6cA

### 拼接字符串的几种方式 
- 通过 + 拼接 `a := "hello" + "world"`
- fmt.Sprintf()
- strings.joins()
- strings.Builder()
- bytes.Builder()

性能：strings.Join ≈ strings.Builder > bytes.Buffer > "+" > fmt.Sprintf

少量字符串拼接，“+”的性能最高

> [go拼接字符串](go拼接字符串.md)

### 怎么验证一个结构体是否实现了接口的所有方法
可以使用go语言的编译器进行，静态类型检查
1. 尝试将这个结构体指针，赋值给对应的接口变量
2. 将nil转化为这个结构体指针，再转化为接口

```go
type Animal interface{
    Eat(food string)
    getAge() uint8s
}

type struct Cat {
}

func main() {
    // 方式一
    var _ Animal = &Cat{}
    // 方式二
    var _ Animal = (*Cat)(nil)
}
```

### defer的执行顺序：先进后出，在return之后执行

### 交换变量 
`a,b = b,a`

### go语言tag的用处
常见于json序列化、db模块对应表字段名称，`gin`框架对应传参字段名
### go怎么解析出tag: 利用反射包reflect

## go中变量的比较
go中slice、map、function这三种类型是无法直接比较的，可以利用 `reflect.DeepEqual()` 比较，另外struct的成员中如果包含这三种类型也不能直接比较。

### 如何比较两个slice
slice不能直接比较，可以利用`reflect.DeepEqual()`，该方法可以用于slice、map、struct等

## 常见关键字
### select
- select是go中的控制结构，类似于switch
- select 语句**只能用于通道操作**，每个 case 必须是一个通道操作，要么是发送要么是接收。
- select 语句会监听所有指定的通道上的操作，一旦其中一个通道准备好就会执行相应的代码块。
- 如果多个通道都准备好，那么 select 语句会随机选择一个通道执行。如果所有通道都没有准备好，那么执行 default 块中的代码。如果没有default，会阻塞等待有通道可以执行

```go
func main() {
	ch := make(chan int)
	select {
	case i := <-ch:
		println(i)
	default:
		println("default")
	}
}
// <-ch 不会造成阻塞，因为可以执行default
```

## channel什么情况会引发panic
1. 关闭为初始化的channel
2. 重复关闭channel
3. 向已经关闭channel发送数据
4. 在向channel发送数据的过程中关闭channel

## 如何判断channel是否关闭

解决方案一：读取channel是使用多值返回，第二可以返回值为false时，表示channel已经关闭

解决访问二：for range 会自动判断channel是否结束，如果结束自动退出循环

> select可以接收关闭的channel消息，得到nil

## 如何优雅的关闭channel
channel关闭守则：
- 应该只在发送端关闭channel
- 如果存在多个发送者时不要关闭发送者channel，而是使用专门的stop channel。（发送者和接收者多对一是，接受者关闭stop channel，多对多是任意一方关闭channel，双方监听stop channel后，及时终止发送和接收）



# 其他
1. 永远不要使用一个指针指向一个**接口类型**，因为它已经是一个指针
2. 当且仅当动态值和动态类型都为 nil 时，**接口类型**值才为 nil
3. for range 使用短变量声明(:=)的形式迭代变量，需要注意的是，变量 i、v 在每次循环体中都会被重用，而不是重新声明，在**使用指针的需要特别注意** >> [例子](http://mian.topgoer.com/%E7%AC%AC%E4%BA%8C%E5%8D%81%E4%B9%9D%E5%A4%A9/)