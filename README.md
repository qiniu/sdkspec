# Qiniu Cloud Storage SDK Specification

[![Qiniu Logo](http://qiniutek.com/images/logo-2.png)](http://qiniu.com/)

## 语言差异性

- 命名风格：不同语言可以有不同的命名风格。本规范主要按类 Golang 风格进行描述（但不完全是）。
- 名字空间：有的语言没有 package（namespace），通常通过名字前缀来表达。
- 构造函数：有的语言没有构造函数，通过 NewXXX 函数来表达。本规范因为按 Golang 风格，构造函数也是用 NewXXX 进行描述。
- 函数重载：有的语言没有函数重载，通过 XXXYYY 形式命名，其中 XXX 是功能，YYY 是不同重载函数的区分段。支持函数重载的语言没有 YYY 段。
- 函数多返回值：有的语言不支持返回多个返回值，也不支持返回元组（tuple）。


## 各个语言的 SDKSpec

- [Objective-C (iOS)](https://github.com/qiniu/sdkspec/blob/develop/ios/README.md)
- [Java (Android)](https://github.com/qiniu/sdkspec/blob/develop/android/README.md)
- [Java](https://github.com/qiniu/sdkspec/blob/develop/java/README.md)
- [PHP](https://github.com/qiniu/sdkspec/blob/develop/php/README.md)
- [Python](https://github.com/qiniu/sdkspec/blob/develop/python/README.md)
- [Ruby](https://github.com/qiniu/sdkspec/blob/develop/ruby/README.md)
- [Node.js](https://github.com/qiniu/sdkspec/blob/develop/nodejs/README.md)
- [C#](https://github.com/qiniu/sdkspec/blob/develop/csharp/README.md)
- [C/C++](https://github.com/qiniu/sdkspec/blob/develop/c/README.md)
- [Go](https://github.com/qiniu/sdkspec/blob/develop/golang/README.md)

