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

type Client struct {
	...
}

func New() Client

func (this Client) Stat(bucket, key string) (Entry, error)
func (this Client) Delete(bucket, key string) (error)
func (this Client) Move(bucketSrc, keySrc, bucketDest, keyDest string) (error)
func (this Client) Copy(bucketSrc, keySrc, bucketDest, keyDest string) (error)

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
	Code  int
	Error string
}
type BatchStatItemRet struct {
	Data  Entry
	Code  int
	Error string
}

func (this Client) BatchStat(entries []EntryPath) ([]BatchStatItemRet, error)
func (this Client) BatchDelete(entries []EntryPath) ([]BatchItemRet, error)
func (this Client) BatchMove(entries []EntryPathPair) ([]BatchItemRet, error)
func (this Client) BatchCopy(entries []EntryPathPair) ([]BatchItemRet, error)
```

范围：仅在服务端使用

## 数据处理API（fop）

```{go}
package "qiniu/api/fop"

type Client struct {
	...
}

func New() Client

// ImageView

type ImageView struct {
	Mode uint		// 1或2
	Width uint		// Width 默认为0，表示不限定宽度
	Height uint		
	Quality uint	// 质量, 1-100
	Format string	// 输出格式, jpg, gif, png, tif 等图片格式
}

func (this *ImageView) MakeRequest(url string) string

// ImageInfo

type ImageInfoRet struct {
	Format string
	Width uint
	Height uint
	ColorModel string
}

type ImageInfo struct {}

func (this ImageInfo) MakeRequest(url string) (string)
func (this ImageInfo) Call(url string) (ImageInfoRet, error)

// ImageMogrify

type ImageMogrify struct {
	AutoOrient bool		// 根据原图EXIF信息自动旋正
	Thumbnail string	// 缩略图尺寸
	Gravity string		// 
	Crop string			// 裁剪尺寸
	Quality uint		// 质量
	Rotate uint			// 旋转角度, 单位为度
	Format string		// png, jpg等图片格式
}

func (this *ImageMogrify) MakeRequest(url string) (string) // 将url和uri合并,生成请求链接

// ImageExif

type ValType struct {
	Val string
	Type int
}

type ImageExifRet struct {
	Model             ValType
	ColorSpace        ValType
	ImageLength       ValType
	YResolution       ValType
	ExifVersion       ValType
	ResolutionUnit    ValType
	FlashPixVersion   ValType
	Software          ValType
	Orientation       ValType
	Make              ValType
	DateTimeOriginal  ValType
	UserComment       ValType
	YCbCrPositioning  ValType
	XResolution       ValType
	ImageWidth        ValType
	DateTime          ValType
	DateTimeDigitized ValType
}

type ImageExif struct {}

func (this ImageExif) MakeRequest(url string) (string)
func (this ImageExif) Call(url string) (ImageExifRet, error)
```
范围：仅在服务端使用
