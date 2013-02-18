# Qiniu Resource (Cloud) Storage SDK Specification

## 语言差异性

- 命名风格：不同语言可以有不同的命名风格。本规范主要按类 Golang 风格进行描述（但不完全是）。
- 名字空间：有的语言没有 package（namespace），通常通过名字前缀来表达。
- 构造函数：有的语言没有构造函数，通过 NewXXX 函数来表达。本规范因为按 Golang 风格，构造函数也是用 NewXXX 进行描述。
- 函数重载：有的语言没有函数重载，通过 XXXYYY 形式命名，其中 XXX 是功能，YYY 是不同重载函数的区分段。支持函数重载的语言没有 YYY 段。
- 函数多返回值：有的语言不支持返回多个返回值，也不支持返回元组（tuple）。


## 服务端配置（conf）

```{go}
package "qiniu/conf"

var RS_HOST string

var ACCESS_KEY string
var SECRET_KEY string
```

范围：仅在服务端使用


## 构造带授权的 HTTP Client（conn）

```{go}
package "qiniu/auth"


// class auth.Client

type Client struct {
    ...
}

```

范围：仅在服务端使用


## API 请求授权（digestAuth）

```{go}
package "qiniu/auth/digest"


// class digest.Client extends auth.Client { ... }

type Client struct {

    auth.Client // extends auth.Client

    ...
}


func New() (conn digest.Client) { ... }
```

范围：仅在服务端使用


## 上传请求授权（uploadAuth）

```{go}
package "qiniu/auth/up"


// class up.Client extends auth.Client { ... }

type Client struct {

    auth.Client // extends auth.Client

    ...
}


func New(uploadToken string) (conn up.Client) { ... }
```

范围：服务端 / 客户端 使用


## 生成上传/下载授权凭证(token)

```{go}
package "qiniu/auth"


// class auth.PutPolicy

type PutPolicy struct {
    Scope string // 可以是 bucketName 或者 bucketName:key
    Expires int64
    CallbackUrl string
    CallbackBodyType string
    Customer string
    Escape bool
    AsyncOps string
    ReturnBody string
}

// 生成 uploadToken

func (this *PutPolicy) Token() (uploadToken string) { ... }


// class auth.GettPolicy

type GetPolicy struct {
    Scope string // 格式是 domainPattern/keyPattern，没有默认值，用 */* 授权粒度过大，用 */key 比较合适。
    Expires int64
}

// 生成 downloadToken

func (this *GetPolicy) Token() (downloadToken string) { ... }

```

范围：仅在服务端使用


## 上传文件（upload）

```{go}
package "qiniu/up"


// class up.Service

type Service struct {

    Conn auth.up.Client

    Host string // 上传地址，一般是 <bucket>.qiniup.com

}


// 上传一个流，有待补充

func (this *Service) Put()


// 上传一个文件，有待补充

func (this *Service) PutFile()


// 断点续上传，有待补充，这个func可能还需要拆分下

func (this *Service) ResumablePut()


func New(conn auth.up.Client, host string) (up up.Service) { ... }

```

范围：服务端 / 客户端 均可使用


## 存储API（rs）

```{go}
package "qiniu/rs"


// class rs.Service

type Service struct {

    Conn auth.digest.Client

}

// 查看单个文件信息

func (this *Service) Stat(entryURI string)


// 删除单个文件

func (this *Service) Delete(entryURI string)


// 复制单个文件

func (this *Service) Copy(entryURISrc, entryURIDest string)


// 移动单个文件

func (this *Service) Move(entryURISrc, entryURIDest string)


// 批量操作

func (this *Service) BatchStat([entryURI, ...])

func (this *Service) BatchDelete([entryURI, ...])

func (this *Service) BatchCopy([[entryURISrc, entryURIDest], ...])

func (this *Service) BatchMove([[entryURISrc, entryURIDest], ...])


func New(conn auth.digest.Client) (rs rs.Service) { ... }

```

范围：仅在服务端使用


## 文件处理（fop）

```{go}
package "qiniu/fop"


func ImageInfo(imageURL string) (data JSON) { ... }

func ImageExif(imageURL string) (data JSON) { ... }

func ImageMogrifyForPreview(imageURL string, mogrifyOptions map[string]string) (previewURL string) { ... }


// class fop.Service

type Service struct {

    Conn auth.digest.Client

}


// 获取文件临时授权下载链接（private, 主要是 saveAs 要用）

func (this *Service) get(entryURI string) (url string, err error) { ... }


// 文件在线处理并持久化存储

func (this *Service) SaveAs(entryURISrc, entryURIDest, opStr string)


// 图像在线处理（缩略、裁剪、旋转、转化）后并持久化存储

func (this *Service) ImageMogrifySaveAs(entryURISrc, entryURIDest string, mogrifyOptions map[string]string)


func New(conn auth.digest.Client) (fop fop.Service) { ... }

```

范围：服务端
