# go学习笔记
重要的事，看官方文档！！！必须看官方文档，不然，坑得你不要不要的

> [go 简易教程](https://learnku.com/docs/the-little-go-book)
> 
> [golang 标准库文档](https://studygolang.com/pkgdoc)
>
> [官网镜像](http://docscn.studygolang.com/)
> 
> [go语言入门教程](http://c.biancheng.net/golang/)
## go 学习资料
- [7 个优质开源的 Go 项目](https://juejin.cn/post/7092788846781267975)
- [go语言101](https://gfw.go101.org/article/101.html)
- [go编程之旅](https://golang2.eddycjy.com/)
- [go语言原本](https://golang.design/under-the-hood/)


## 安装GO

OSX系统可以从 [golang官网](https://golang.org/) 下载安装包安装

Go 被设计成代码在工作区内运行。工作区是一个文件夹，这个文件夹由 `bin` ，`pkg`，以及 `src` 子文件夹组成的

设置两个环境变量：

1. GOPATH 指向的是你的工作目录
2. 我们需要将 Go 的二进制文件添加到的 PATH 变量中。

你可以通过下面的 shell 去设置这两个环境变量：
```
echo 'export GOPATH=$HOME/go' >> $HOME/.profile
echo 'export PATH=$PATH:/usr/local/go/bin' >> $HOME/.profile
```

你需要将这些环境变量激活。你可以关掉 shell 终端，然后在打开 shell 终端，或者你可以在 shell 终端运行 `source $HOME/.profile`

输入 `go version` 验证是否安装成功

---

Go **不是** 像 C ++，Java，Ruby 和 C＃一样的面向对象的（OO）语言。它没有对象和继承的概念，也没有很多与面向对象相关的概念，例如多态和重载。

Go 所具有的是结构体的概念，可以将一些方法和结构体关联。Go 还支持一种简单但有效的组合形式。 总的来说，它会使代码变的更简单，但在某一些场合，你会错过面向对象提供的一些特性

 ---

## 运行go文件的方式
hello.go
1. `go run hello.go`
2. 生成可执行文件
```
go build hello.go
## 会生成一个hello的可执行文件
./hello 
```

## 变量

作用域

- 函数内定义的变量称为**局部变量**
- 函数外定义的变量称为**全局变量**
- 函数定义中的变量称为**形参变量**

### 数组
具有**相同数据类型**的一组**已编号**的且**长度固定**的数据项。

```
var week [7] = {0,1,2,3,4,5,6}
```

### 切片Slice
在go语言中我们很少直接使用数组，取而代之的是 **切片** 又称为 **动态数组** ，相对于数组他的长度是可变的，可以追加元素。**slice的底层是数组**指针，因此copy出来的slice修改，会互相影响，在发生扩容之后会生成新的底层数据，其他之间的影响才会切断

```go
package main

import "fmt"
//切片有称为动态数组，相对于数组而言，它的长度是不固定的，可以动态的追加和删除
func main() {
	//初始化一个切片
	s := []int{1,2,3}
	
	fmt.Println(s)
	fmt.Printf("len:%d, cap:%d, s:%v\n", len(s), cap(s), s)
	
	//初始化一个长度为3，容量为5的切片
	k := make([]int, 3, 5)
	fmt.Println(k)

	fmt.Println(s[1:])
	fmt.Println(s[:])
	fmt.Println(s[:2])

	//像切片中追加元素，注意数据类型必须一样
	s = append(s, 4)
	s = append(s, 5, 6) //可以同时追加多个
	fmt.Println(s)
}
```

### 集合map
是一种 **无序** 的 键值对集合，通过key可以快速检索数据（底层由**hash表**实现）

range map输出的顺序是不确定的

### 结构体 struct
可以为不同项定义不同的数据类型。结构体是由一组相同的或不同类型的数据构成的数据集合

## 关键字range
用于for循环中迭代数组（array）、切片（slice）、通道（channel）或集合（map）的元素

---

## package
- package 是最基本的分发单位和工程管理中依赖关系的体现；
- 每个go语言源码文件 **开头** 都拥有一个package声明，表示源码文件所属代码包
- 要生成go **可执行程序** ，必须有main的package包，且必须再改包下有 `main()` 函数
- **同一个路径下只能存在一个package** ，一个package可以拆成多个源文件
- 方法名首字母 **大写** ，表示开放方法，相当于`public`声明，相反的首字母 **小写** 意味着是`private`方法


## 其他教程
> https://github.com/astaxie/build-web-application-with-golang/blob/master/zh/preface.md


# Golang新开发者要注意的陷阱和常见错误

> 译文：[Go的50度灰](https://colobu.com/2015/09/07/gotchas-and-common-mistakes-in-go-golang/#%E5%88%9D%E7%BA%A7)
> 
> 原文：http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/