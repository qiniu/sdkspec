# Qiniu Resource (Cloud) Storage SDK Specification

## 语言差异性

- 命名风格：不同语言可以有不同的命名风格。本规范主要按类 Golang 风格进行描述（但不完全是）。
- 名字空间：有的语言没有 package（namespace），通常通过名字前缀来表达。
- 构造函数：有的语言没有构造函数，通过 NewXXX 函数来表达。本规范因为按 Golang 风格，构造函数也是用 NewXXX 进行描述。
- 函数重载：有的语言没有函数重载，通过 XXXYYY 形式命名，其中 XXX 是功能，YYY 是不同重载函数的区分段。支持函数重载的语言没有 YYY 段。


## 生成上传/下载凭证(token)

```{go}
package qiniu.auth

class PutPolicy {
	scope string // 可以是 bucketName 或者 bucketName:key
	callbackUrl string
	callbackBodyType string
	customer string
	syncOps string
	expires int
	escape bool
}

func PutPolicy.token() string

class GetPolicy {
	scope string // 格式是 domainPattern/keyPattern，没有默认值，用 */* 授权粒度过大，用 */key 比较合适。
	expires int
}

func GetPolicy.token() string
```

范围：仅在服务端使用


## API请求授权(digestauth)

```{go}
package qiniu.digestauth

class Client {
	...
}

func New() Client
```

范围：仅在服务端使用


## 上传授权(uploadauth)

```{go}
package qiniu.uploadauth

class Client {
	...
}

func New(uptoken string) Client
```

范围：服务端或客户端


## 普通上传(upload)


## 断点续上传(resumable upload)


## 存储API(rs)

```{go}
package qiniu.rs

class Service {
}
```

范围：仅在服务端使用


## 文件处理(filop)

```{go}
package qiniu.fileop
```

范围：服务端或客户端

