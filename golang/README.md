# Golang SDK Specification

[![Qiniu Logo](http://qiniutek.com/images/logo-2.png)](http://qiniu.com/)

## 服务端配置（conf）

```{go}
package "qiniu/api/conf"

var USER_AGENT string  // 请求的 User-Agent 值，比如 "qiniu php-sdk v6.0.0"

var UP_HOST  string
var RS_HOST  string
var RSF_HOST string

var ACCESS_KEY string
var SECRET_KEY string  // 不要在客户端初始化该变量
```

范围：服务端和客户端共用


## 签名认证（auth/digest）

```{go}
package "qiniu/api/auth/digest"

type Mac struct {
	AccessKey string
	SecretKey []byte
}
```

范围：仅在服务端使用


## 存储API（rs）

```{go}
package "qiniu/api/rs"

type Client struct {
	...
}

func New(mac *digest.Mac = nil) Client

func (this Client) Stat(bucket, key string) (entry Entry, err error)
func (this Client) Delete(bucket, key string) (err error)
func (this Client) Move(bucketSrc, keySrc, bucketDest, keyDest string) (err error)
func (this Client) Copy(bucketSrc, keySrc, bucketDest, keyDest string) (err error)

type Entry struct {
	Hash     string
	Fsize    int64
	PutTime  int64
	MimeType string
	EndUser  string
}

// batch

type EntryPath struct {
	Bucket string
	Key    string
}

type EntryPathPair struct {
	Src  EntryPath
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


## 上传/下载授权凭证（uptoken/dntoken）

```{go}
package "qiniu/api/rs"

type PutPolicy struct {
	Scope	     string  // 必选。可以是 bucketName 或者 bucketName:key
	CallbackUrl  string  // 可选。
	CallbackBody string  // 可选
	ReturnUrl    string  // 可选
	ReturnBody   string  // 可选
	AsyncOps     string  // 可选
	EndUser	     string  // 可选
	Expires	     uint32  // 可选。默认是 3600 秒
}

// 通常只有在key为非utf编码序列时候，才会返回错误
func (this *PutPolicy) Token(mac *digest.Mac = nil) (uptoken string， err error)

type GetPolicy struct {
	Expires uint32  // 可选。默认是 3600 秒
}

func (this GetPolicy) MakeRequest(baseUrl string, mac *digest.Mac = nil) (privateUrl string)

func MakeBaseUrl(domain, key string) (baseUrl string)
```

范围：仅在服务端使用


## 存储高级API（rsf）

```{go}
package "qiniu/api/rsf"

type Client struct {
	...
}

func New(mac *digest.Mac = nil) Client

func (this Client) ListPrefix(
	bucket, prefix, marker string, limit int) (entries []ListItem, markerOut string, err error)
	// 1. 首次请求 marker = ""
	// 2. 无论 err 值如何，均应该先看 entries 是否有内容
	// 3. 如果后续没有更多数据，err 返回 EOF，markerOut 返回 ""（但不通过该特征来判断是否结束）

type ListItem struct {
	Key      string
	Hash     string
	Fsize    int64
	PutTime  int64
	MimeType string
	EndUser  string
}
```

这个 `ListPrefix` 的标准用法如下：

```{go}
import "qiniu/api/rsf"

func listAll(rs rsf.Client, bucket, prefix string, limit int) {

	var items []rsf.ListItem
	var marker string
	var err error
	for err == nil {
		items, marker, err = rs.ListPrefix(bucket, prefix, marker, limit)
		for _, item := range items {
			... // 处理item
		}
	}
	if err != rsf.EOF {
		... // 错误处理
	}
}
```

范围：仅在服务端使用


## 上传（io）

```{go}
package "qiniu/api/io"

type PutExtra struct {
	Params	 map[string]string  // 用户自定义参数，key必须以 "x:" 开头
	MimeType string             // 可选
	Crc32	 uint32
	CheckCrc uint32
		// CheckCrc == 0: 表示不进行 crc32 校验
		// CheckCrc == 1: 对于 Put 等同于 CheckCrc = 2；对于 PutFile 会自动计算 crc32 值
		// CheckCrc == 2: 表示进行 crc32 校验，且 crc32 值就是上面的 Crc32 变量
}

// 如果 uptoken 没有指定 ReturnBody，那么返回值是标准的 PutRet 结构
type PutRet struct {
	Hash string
	Key  string
}

func Put(
	ret interface{}, uptoken string, key string, body io.Reader, extra *PutExtra) (err error)

func PutWithoutKey(
	ret interface{}, uptoken string, body io.Reader, extra *PutExtra) (err error)

func PutFile(
	ret interface{}, uptoken string, key string, localFile string, extra *PutExtra) (err error)

func PutFileWithoutKey(
	ret interface{}, uptoken string, localFile string, extra *PutExtra) (err error)
```

范围：客户端和服务端


## 断点续上传（resumable io）

```{go}
package "qiniu/api/resumable/io"

// upload

type BlkputRet struct {
	Ctx      string  `json:"ctx"`
	Checksum string  `json:"checksum"`
	Crc32    uint32  `json:"crc32"`
	Offset   uint32  `json:"offset"`
}

type PutExtra struct {
	Params	   map[string]string  // 用户自定义参数，key必须以 "x:" 开头
	MimeType   string             // 可选。若为"" 则服务端自动判断
	ChunkSize  int                // 可选。每次上传的Chunk大小
	TryTimes   int                // 可选。尝试次数
	Progresses []BlkputRet        // 可选。上传进度
	Notify	   func(blkIdx int, blkSize int, ret *BlkputRet)  // 进度提示。注意blk是并行传输的
	NotifyErr  func(blkIdx int, blkSize int, err error)
}

// 如果 uptoken 没有指定 ReturnBody，那么返回值是标准的 PutRet 结构
type PutRet struct {
	Hash string
	Key  string
}

func Put(
	ret interface{}, uptoken string,
	key string, f io.ReaderAt, fsize int64, extra *PutExtra) (err error)

func PutWithoutKey(
	ret interface{}, uptoken string, f io.ReaderAt, fsize int64, extra *PutExtra) (err error)

func PutFile(
	ret interface{}, uptoken string, key string, localFile string, extra *PutExtra) (err error)

func PutFileWithoutKey(
	ret interface{}, uptoken string, localFile string, extra *PutExtra) (err error)

func BlockCount(fsize int64) int

// global settings

type Settings {
	TaskQsize int  // 可选。任务队列大小。为 0 表示取 Workers * 4
	Workers   int  // 并行的工作线程数目。
	ChunkSize int  // 默认的Chunk大小，不设定则为256k
	TryTimes  int  // 默认的尝试次数，不设定则为3
}

func SetSettings(settings *Settings)
```

范围：客户端和服务端


## 数据处理API（fop）

```{go}
package "qiniu/api/fop"

// imageView

type ImageView struct {
	Mode    int     // 缩略模式
	Width   int     // Width = 0 表示不限定宽度
	Height  int     // Height = 0 表示不限定高度
	Quality int     // 质量, 1-100
	Format  string  // 输出格式，如jpg, gif, png, tif等等
}

func (this *ImageView) MakeRequest(url string) (imageViewUrl string)

// imageMogr

type ImageMogrify struct {
	...				// 待标准化
}

func (this *ImageMogrify) MakeRequest(url string) (imageMogrUrl string)

// imageInfo

type ImageInfoRet struct {
	Width      int
	Height     int
	Format     string
	ColorModel string
}

type ImageInfo struct {}

func (this ImageInfo) MakeRequest(url string) (imageInfoUrl string)

// exif

type Exif struct {}

func (this Exif) MakeRequest(url string) (imageExifUrl string)
```

范围：客户端和服务端


## 服务端上传（rsutil）

```{go}
package "qiniu/api/rsutil"

import (
	"qiniu/api/rs"
	sio "qiniu/api/io"
	rio "qiniu/api/resumable/io"
)

// simple upload

func Put(
	c rs.Client, bucket string,
	key string, body io.Reader, extra *sio.PutExtra) (ret sio.PutRet, err error)

func PutWithoutKey(
	c rs.Client, bucket string,
	body io.Reader, extra *sio.PutExtra) (ret sio.PutRet, err error)

func PutFile(
	c rs.Client, bucket string,
	key string, localFile string, extra *sio.PutExtra) (ret sio.PutRet, err error)

func PutFileWithoutKey(
	c rs.Client, bucket string,
	localFile string, extra *sio.PutExtra) (ret sio.PutRet, err error)

// resumable upload

func Rput(
	c rs.Client, bucket string,
	key string, f io.ReaderAt, fsize int64, extra *rio.PutExtra) (ret rio.PutRet, err error)

func RputWithoutKey(
	c rs.Client, bucket string,
	f io.ReaderAt, fsize int64, extra *rio.PutExtra) (ret rio.PutRet, err error)

func RputFile(
	c rs.Client, bucket string,
	key string, localFile string, extra *rio.PutExtra) (ret rio.PutRet, err error)

func RputFileWithoutKey(
	c rs.Client, bucket string,
	localFile string, extra *rio.PutExtra) (ret rio.PutRet, err error)
```
范围：仅在服务端使用。本模块仅仅是 rs + io/rio 的简单包装。

## 错误处理规范
1. 原则上`err`不应当跨`package`直接传递，比如 `api/io.put`调用`rpc`包的某个函数产生错误，不应该直接`return err`，可以适当的封装点一些信息比如 `err = fmt.Errorf("rpc net %v",  err)`
2. 在适当的层级可以对`err`隐藏隔离，定义具有自己意义的错误，而没必要一味的返回，比如`PutPolicy.Token`中的 `json.Unmarshal`时候可能遇到错误`json.InvalidUTF8Error`。`io.Put`通常也会遇到非`utf8 key`, key中包含`\n`等问题。这种情况可以统一定义个`EINVALIDKEY = errors.New("invalid key format")`。

## 单元测试规范
1. 测试环境的初始化和清理工作应当看做单元测试的一部分。
2. SDK对外的接口都应该覆盖到，测试部分的代码风格和质量也要得到保证。
