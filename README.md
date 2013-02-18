# Qiniu Resource (Cloud) Storage SDK Specification

## 语言差异性

- 命名风格：不同语言可以有不同的命名风格。本规范主要按类 Golang 风格进行描述（但不完全是）。
- 名字空间：有的语言没有 package（namespace），通常通过名字前缀来表达。
- 构造函数：有的语言没有构造函数，通过 NewXXX 函数来表达。本规范因为按 Golang 风格，构造函数也是用 NewXXX 进行描述。
- 函数重载：有的语言没有函数重载，通过 XXXYYY 形式命名，其中 XXX 是功能，YYY 是不同重载函数的区分段。支持函数重载的语言没有 YYY 段。
- 函数多返回值：有的语言不支持返回多个返回值，也不支持返回元组（tuple）。


## 服务端配置（conf）

```
package qiniu.conf

var RS_HOST string = "http://rs.qbox.me"

var ACCESS_KEY string
var SECRET_KEY string
```

范围：仅在服务端使用


## 构造带授权的 HTTP Client（conn）

```
package qiniu.auth

class Client {
    func auth(opts, body) // 可重载
    func execute(...)
    func callWith(...)
}

```

范围：仅在服务端使用


## API 请求授权（digestAuth）

```
package qiniu.auth.rs

class rs.Client extends auth.Client {

    func auth(opts, body) // 支持 mac token 的 digest auth 代码

    ...
}

func New() (conn rs.Client) {}
```

范围：仅在服务端使用


## 上传请求授权（uploadAuth）

```
package qiniu.auth.up

class up.Client extends auth.Client {

    func auth(opts, body) // 支持 upload token 的代码

    ...

}

func New(uploadToken string) (conn up.Client) {}
```

范围：服务端 / 客户端 使用


## 下载请求授权（downloadAuth）

```
package qiniu.auth.dn

class dn.Client extends auth.Client {

    func auth(opts, body) // 支持 download token 的代码

    ...
}

// downloadToken 允许为空，指下载公有资源
func New(downloadToken string) (conn dn.Client) { … }
```

范围：服务端 / 客户端 使用


## 生成上传/下载授权凭证(token)

```
package qiniu.auth

class PutPolicy {
    scope string // 可以是 bucketName 或者 bucketName:key
    callbackUrl string
    callbackBodyType string
    customer string
    syncOps string
    expires int
    escape bool

    // 生成 uploadToken
    func token() (uploadToken string) { … }
}


class GetPolicy {
    scope string // 格式是 domainPattern/keyPattern，没有默认值，用 */* 授权粒度过大，用 */key 比较合适。
    expires int

    // 生成 downloadToken
    func token() (downloadToken string) { … }
}
```

范围：仅在服务端使用


## 上传文件（upload）

```
package qiniu.up

class Service {

    Conn auth.up.Client

    Host string // 上传地址，一般是 <bucket>.qiniup.com


    // 上传一个流，有待补充
    func putStream()

    // 上传一个文件，有待补充
    func putFile()

    // 断点续上传，有待补充，这个func可能还需要拆分下
    func resumablePut()
}

func New(conn auth.up.Client, host string) (up up.Service) { … }

```

范围：服务端 / 客户端 均可使用


## 下载文件（download）

```
package qiniu.dn

class Service {

    Conn auth.dn.Client

    Host string // 下载地址，一般是 <bucket>.qiniudn.com

    // 获取下载链接
    func get(key string) (url string) { … }

    // 下载文件
    func download(key string) (code int, body []byte, err error) { … }

    // 断点续下载
    func downloadByRange(key string, firstBytePos, lastBytePos int64) (code int, body []byte, err error) { … }
}

func New(conn auth.dn.Client, host string) (dn dn.Service) { … }
```

范围：服务端 / 客户端 均可使用


## 存储API（rs）

```
package qiniu.rs

class Service {

    Conn auth.rs.Client

    // 查看单个文件信息
    func Stat(entryURI string)

    // 删除单个文件
    func Delete(entryURI string)

    // 复制单个文件
    func Copy(entryURISrc, entryURIDest string)

    // 移动单个文件
    func Move(entryURISrc, entryURIDest string)

    // 批量操作
    func BatchStat([entryURI, …])

    func BatchDelete([entryURI, …])

    func BatchCopy([[entryURISrc,entryURIDest], …])

    func BatchMove([[entryURISrc,entryURIDest], …])
}

func New(conn auth.rs.Client) (rs rs.Service) { … }
```

范围：仅在服务端使用


## 文件处理(fop)

```
package qiniu.fop


func ImageInfo(url string) (data JSON) { … }

func ImageExif(url string) (data JSON) { … }

func ImageMogrifyPreview(srcImageUrl string, mogrifyOptions map[string]string) (previewURL string) { … }



class Service {

    Conn auth.rs.Client

    // 获取文件临时授权下载链接（private, 主要是 saveAs 要用）
    func get(entryURI string)

    // 文件在线处理并持久化存储
    func SaveAs(entryURISrc, entryURIDest, opStr string)

    // 图像在线处理（缩略、裁剪、旋转、转化）后并持久化存储
    func ImageMogrifySaveAs(entryURISrc, entryURIDest, mogrifyOptions map[string]string)

}

func New(conn auth.rs.Client) (fop fop.Service) { … }

```

范围：服务端

