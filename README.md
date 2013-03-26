# Qiniu Resource (Cloud) Storage SDK Specification

[![Qiniu Logo](http://qiniutek.com/images/logo-2.png)](http://qiniu.com/)

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
	Escape uint16				// 可选。非 0 表示 Callback 的 Params 支持转义符
	DetectMime uint16			// 可选。非 0 表示在服务端自动检测文件内容的 MimeType
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

type Client struct {
	...
}

func New() Client

func (this Client) Stat(bucket, key string) (entry Entry, err error)
func (this Client) Delete(bucket, key string) (err error)
func (this Client) Move(bucketSrc, keySrc, bucketDest, keyDest string) (err error)
func (this Client) Copy(bucketSrc, keySrc, bucketDest, keyDest string) (err error)

type Entry struct {
	Hash     string
	Fsize    int64
	PutTime  int64
	MimeType string
	Customer string
}

// batch

type EntryPath struct {
	Bucket string
	Key string
}

type EntryPathPair struct {
	Src EntryPath
	Dest EntryPath
}

type BatchItemRet struct {
	Error string
	Code  int
}

type BatchStatItemRet struct {
	Data  Entry
	Error string
	Code  int
}

func (this Client) BatchStat(entries []EntryPath) (rets []BatchStatItemRet, err error)
func (this Client) BatchDelete(entries []EntryPath) (rets []BatchItemRet, err error)
func (this Client) BatchMove(entries []EntryPathPair) (rets []BatchItemRet, err error)
func (this Client) BatchCopy(entries []EntryPathPair) (rets []BatchItemRet, err error)
```

范围：仅在服务端使用


## 上传/下载（io）

```{go}
package "qiniu/api/io"

// upload

type PutExtra struct {
	CallbackParams	string	// 当 uptoken 指定了 CallbackUrl，则 CallbackParams 必须非空
	CustomMeta		string	// 可选。用户自定义 Meta，不能超过 256 字节
	MimeType		string	// 可选。在 uptoken 没有指定 DetectMime 时，用户客户端可自己指定 MimeType
}

type PutRet struct {
	Hash			string	// 如果 uptoken 没有指定 ReturnBody，那么返回值是标准的 PutRet 结构 
}

func Put(ret interface{}, uptoken string, bucket, key string, body io.Reader, fsize int64, extra *PutExtra) (err error)
func PutFile(ret interface{}, uptoken string, bucket, key string, localFile string, extra *PutExtra) (err error)

// download

func GetUrl(domain string, key string, dntoken string) (downloadUrl string)
```

范围：客户端和服务端


## 断点续上传（up）

```{go}
package "qiniu/api/up"

todo :-)
```

范围：客户端和服务端


## 数据处理API（fop）

```{go}
package "qiniu/api/fop"

// imageView

type ImageView struct {
	Mode int		// 缩略模式
	Width int		// Width = 0 表示不限定宽度
	Height int		// Height = 0 表示不限定高度
	Quality int		// 质量, 1-100
	Format string	// 输出格式，如jpg, gif, png, tif等等
}

func (this *ImageView) MakeRequest(url string) (imageViewUrl string)

// imageMogr

type ImageMogrify struct {
	...				// 待标准化
}

func (this *ImageMogrify) MakeRequest(url string) (imageMogrUrl string)

// imageInfo

type ImageInfoRet struct {
	Width int
	Height int
	Format string
	ColorModel string
}

type ImageInfo struct {}

func (this ImageInfo) MakeRequest(url string) (imageInfoUrl string)
func (this ImageInfo) Call(url string) (ret ImageInfoRet, err error)

// exif

type ExifValType struct {
	Val string
	Type int
}

type ExifRet map[string] ExifValType
type Exif struct {}

func (this Exif) MakeRequest(url string) (imageExifUrl string)
func (this Exif) Call(url string) (ret ExifRet, err error)
```

范围：客户端和服务端

