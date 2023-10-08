## go 细节

1. 永远不要使用一个指针指向一个**接口类型**，因为它已经是一个指针
2. 当且仅当动态值和动态类型都为 nil 时，**接口类型**值才为 nil
3. go 中不同类型是不能比较的，而数组长度是数组类型的一部分，所以 […]int{1} 和 [2]int{1} 是两种不同的类型，不能比较；
4. **切片是不能比较**的，可以用`reflect.DeepEqual()`比较
5. for range 使用短变量声明(:=)的形式迭代变量，需要注意的是，变量 i、v 在每次循环体中都会被重用，而不是重新声明，在**使用指针的需要特别注意** >> [例子](http://mian.topgoer.com/%E7%AC%AC%E4%BA%8C%E5%8D%81%E4%B9%9D%E5%A4%A9/)


### map 的赋值操作
对于类似 X = Y的赋值操作，必须知道 X 的地址，才能够将 Y 的值赋给 X，但 go 中的 map 的 value 本身是不可寻址的，因此value不可修改，只能整体替换
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


### new 与 make的区别
new与make都会分配内存

- new只用于分配内存，返回一个**指向地址的指针**。它为每个新类型分配一片内存，初始化为0且返回类型*T的内存地址，它相当于`&T{}`
- make只可用于`slice`,`map`,`channel`的初始化，返回的所创建类型本身，而非指针。make初始化时，必须执行长度，即使长度为0

需要注意的是，new函数返回的是指向新分配的**零值内存的指针**，并不会初始化内存中的值。如果需要对分配的内存进行初始化，可以使用make函数（适用于切片、映射和通道类型）或结构体字面量的方式进行初始化。

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
    getAge() uint8
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
### 交换变量 `a,b = b,a`
### go语言`tag`的用处
常见于json序列化、db模块对应表字段名称，`gin`框架对应传参字段名
### go怎么解析出tag: 利用反射包`reflect`
### 如何比较两个slice
slice不能直接比较，可以利用`reflect.DeepEqual()`，该方法可以用于slice、map、struct等