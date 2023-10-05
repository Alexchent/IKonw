# golang 常用包 strings

## 字符串操作函数包 `strings`

- Contains 字符串中是否包含

`fmt.Println(strings.Contains("hellohe", "he"));`
    
- Join 切片合并成一个字符串

`fmt.Println(strings.Join([]string{"a","b"},","));`
    
- Index(s, sep string) int 在字符串s中查找sep第一次出现的位置，找不到返回-1

`fmt.Println(strings.Index("hello", "l"))`
    
- Repeat

`fmt.Println(strings.Repeat("hello", 3))`
    
- Replace 第四个参数表示替换次数，0表示全部

`fmt.Println(strings.Replace("hello world", "hello", "hi", 1))`
    
- Split 字符串分解为切片

`fmt.Println(strings.Split("hello world", " "))`
    
- Trim(s, cutset string) string 字符串s中清除cutstr并返回新的字符串

`fmt.Println(strings.Trim("   hello world   ", " "))`
    
- Fields 清除字符串中的空格，并按空格分割返回slice

`fmt.Println(strings.Fields("   hello world   "))`
