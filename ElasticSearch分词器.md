# ElasticSearch 分词器


> [ElasticSearch最全分词器比较及使用方法](https://blog.csdn.net/ZYC88888/article/details/83620572)

## standard 分词器 *
**支持中文**采用的方法为单字切分。他会将词汇单元转换成小写形式，并去除停用词和标点符号

## simple 分词器 *
首先会通过非字母字符来分割文本信息，然后将词汇单元统一为小写形式。该分析器会去掉数字类型的字符。

## whitespace 分词器
仅仅是去除空格，对字符没有lowcase化,**不支持中文**； 并且不对生成的词汇单元进行其他的规范化处理。

## keyword 分词器 *
把整个输入作为一个单独词汇单元，方便特殊类型的文本进行索引和检索

## stop 分词器
在SimpleAnalyzer的基础上增加了去除英文中的常用单词（如the，a等），也可以增加自己的需要设置常用单词；**不支持中文**

## pattern 分词器
可以通过正则表达式将文本分成"terms"(经过token Filter 后得到的东西 

一个 pattern analyzer 可以做如下的属性设置:

lowercaseterms是否是小写. 默认为 true 小写.pattern正则表达式的pattern, 默认是 \W+.flags正则表达式的flagsstopwords一个用于初始化stop filter的需要stop 单词的列表.默认单词是空的列表

## language 分词器
一个用于解析特殊语言文本的analyzer集合。**不支持中文**

## custom 分词器
自定义分词器，允许多个零到多个**tokenizer**，零到多个 **Char Filters**. custom analyzer 的名字不能以 "_"开头.

# 中文分词器
## IK
IKAnalyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包。需要安装插件