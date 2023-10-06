# go 拼接字符串的方式
- 通过 + 拼接 `a := "hello" + "world"`
- fmt.Sprintf()
- strings.joins()
- strings.Builder()
- bytes.Builder()

性能：strings.Join ≈ strings.Builder > bytes.Buffer > "+" > fmt.Sprintf

示例
```go
package main

func main() {

}
```