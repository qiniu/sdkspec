# Qiniu Resource (Cloud) Storage SDK Specification

## 语言差异性

- 命名风格：不同语言可以有不同的命名风格。本规范主要按类 Golang 风格进行描述（但不完全是）。
- 名字空间：有的语言没有 package（namespace），通常通过名字前缀来表达。
- 构造函数：有的语言没有构造函数，通过 NewXXX 函数来表达。本规范因为按 Golang 风格，构造函数也是用 NewXXX 进行描述。
- 函数重载：有的语言没有函数重载，通过 XXXYYY 形式命名，其中 XXX 是功能，YYY 是不同重载函数的区分段。支持函数重载的语言没有 YYY 段。
- 函数多返回值：有的语言不支持返回多个返回值，也不支持返回元组（tuple）。


## 服务端配置（conf）

```{go}
package "qiniu/api/conf"

var IO_HOST string // 客户端不需要？答：由于未来推荐用户上传时用 <bucket>.qiniup.com，所以没有全局 IO_HOST
var RS_HOST string

var ACCESS_KEY string
var SECRET_KEY string
```

范围：仅在服务端使用


## 授权（auth）

```{go}
package "qiniu/api/auth"

type Client interface {
	...
}
```

范围：服务端或客户端


## API请求授权（digest auth，适用于所有API）

```{go}
package "qiniu/api/auth/digest"

type Client struct {
	...
}

func New() Client
```

范围：仅在服务端使用


## 上传请求授权（upload auth，仅适用于上传API）

```{go}
package "qiniu/api/auth/up"

type Client struct {
	...
}

func New(uptoken string) Client
```

范围：服务端或客户端


## 生成上传/下载授权凭证（uptoken/dntoken）

```{go}
package "qiniu/api/rs"

type PutPolicy struct {
	Scope string				// 必选项。可以是 bucketName 或者 bucketName:key
	CallbackUrl string			// 可选
	CallbackBodyType string		// 可选
	Customer string				// 可选
	AsyncOps string				// 可选
	ReturnBody string			// 可选
	Expires uint32				// 可选。默认是 3600 秒
	Escape bool					// 可选
	DetectMime bool				// 可选
}

func (this *PutPolicy) Token() (uptoken string)

type GetPolicy struct {
    Scope string				// 格式是 domainPattern/keyPattern，没有默认值，用 */* 授权粒度过大，用 */key 比较合适。
    Expires uint32				// 可选。默认是 3600 秒
}

func (this *GetPolicy) Token() (dntoken string)
```

范围：仅在服务端使用


## 存储API（rs）

```{go}
package "qiniu/api/rs"

todo (参考 https://github.com/qiniu/api/rs 定出规范)
```

## 数据处理API（fop）

```{go}
package "qiniu/api/fop"

todo (参考 https://github.com/qiniu/java-sdk/tree/develop/src/main/java/com/qiniu/qbox/fileop 定出规范)
```

