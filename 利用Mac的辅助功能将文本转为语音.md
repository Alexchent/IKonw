# 利用Mac的辅助功能将文本转为语音

### 演示
1. 打开在终端输入 `say`

```
say
告诉老默我想吃鱼了
```


### 朗读文本内容
创建一个txt文本`vim huaben.txt`
随便输入一些想要转化成语音的内容
```
告诉老默我想吃鱼了，风浪越大鱼越贵
金陵岂是池中物
```
读取文本内容 `say -f huaben.txt`

### 将文本内容转化成MP3
```
say -f test.txt -o output.acc

lame output.acc hello.mp3
```


## 查看支持的语音
```
say -v '?'
```
![](assets/17024531101387.jpg)
选择一种声音
```
say -v Panpan 张楚岚，看球
```