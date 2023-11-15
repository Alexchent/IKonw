# go 拼接字符串的方式
- 通过 + 拼接 `a := "hello" + "world"`
- fmt.Sprintf()
- strings.Join()
- strings.Builder()
- bytes.Builder()

性能：strings.Join ≈ strings.Builder > bytes.Buffer > "+" > fmt.Sprintf

少量字符串拼接，“+”的性能最高
示例
```go
func main() {
	// 字符串拼接的几种方式
	// 1. +
	// 2. fmt.Sprintf
	// 3. strings.Join
	// 4. bytes.Buffer
	// 5. strings.Builder
	// 官方推荐 strings.Builder 无论字符串的数据量多少，性能都比较优秀

	a := []string{"a", "b", "c"}
	//方式1：+
	ret := a[0] + a[1] + a[2]
	//方式2：fmt.Sprintf
	ret := fmt.Sprintf("%s%s%s", a[0], a[1], a[2])

	//方式3：strings.Builder
	var sb strings.Builder
	sb.WriteString(a[0])
	sb.WriteString(a[1])
	sb.WriteString(a[2])
	ret := sb.String()

	//方式4：bytes.Buffer
	buf := new(bytes.Buffer)
	buf.Write(a[0])
	buf.Write(a[1])
	buf.Write(a[2])
	ret := buf.String()

	//方式5：strings.Join
	ret := strings.Join(a, "")
	fmt.Println(ret)
}
```