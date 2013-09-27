# Qiniu Cloud Storage PHP SDK Specification

[![Qiniu Lophp](http://qiniutek.com/images/logo-2.png)](http://qiniu.com/)

## 服务端配置（conf）

``` 
{php}
//资源上传服务器地址
$QINIU_UP_HOST	= 'http://up.qiniu.com';

//资源管理服务器地址
$QINIU_RS_HOST	= 'http://rs.qbox.me';

//资源列表服务器地址
$QINIU_RSF_HOST	= 'http://rsf.qbox.me';

//公钥
$QINIU_ACCESS_KEY	= '<Please apply your access key>';

//私钥 不要在客户端初始化该变量
$QINIU_SECRET_KEY	= '<Dont send your secret key to anyone>';

```


## 签名认证（auth digest）

```
{php}

class Qiniu_Mac {

	public $AccessKey;
	public $SecretKey;
	
	public function Sign ($data) {} 
	public function SignWithData ($data) {}
	public function SignRequest ($req, $incbody) {} 
}

```

范围：仅在服务端使用


## http客户端（http）

```

class Qiniu_Error
{
	/**
	 * @var string
	 */
	public $Err;	 // string
	
	/**
	 * @var string
	 */
	public $Reqid;	 // string
	
	/**
	 * @var []string
	 */
	public $Details; // []string
	
	/**
	 * @var int
	 */
	public $Code;	 // int

	public function __construct($code, $err)
	{
		$this->Code = $code;
		$this->Err = $err;
	}
}

class Qiniu_Request
{
	/**
	 * @var string
	 */
	public $URL;
	
	/**
	 * @var []string
	 */
	public $Header;
	
	/**
	 * @var mixed
	 */
	public $Body;

	public function __construct($url, $body)
	{
		$this->URL = $url;
		$this->Header = array();
		$this->Body = $body;
	}
}


class Qiniu_Response
{
	/**
	 * @var int 状态码
	 */
	public $StatusCode;
	
	/**
	 * @var []string
	 */
	public $Header;
	
	/**
	 * @var int 
	 */
	public $ContentLength;
	
	/**
	 * @var mixed
	 */
	public $Body;

	public function __construct($code, $body)
	{
		$this->StatusCode = $code;
		$this->Header = array();
		$this->Body = $body;
		$this->ContentLength = strlen($body);
	}
}

/**
 * httpclient
 */
class Qiniu_HttpClient
{
	/**
	 * @param Qiniu_Request $req
	 * @return array(Qiniu_Response $resp, Qiniu_Error $error)
	 */
	public function RoundTrip($req) {}
}

/**
 * 经过认证的httpclient
 */
class Qiniu_MacHttpClient
{
	/**
	 * @var Qiniu_Mac
	 */
	public $Mac;

	public function __construct($mac) {}

	/**
	 * @param Qiniu_Request $req
	 * @return array(Qiniu_Response $resp, Qiniu_Error $error)
	 */
	public function RoundTrip($req) {}
}

```



## 存储API（rs）

```
{php}

/**
 * 资源路径: $bucket . ':' . $key 
 */
class Qiniu_RS_EntryPath
{
	/**
	 * @var string Bucket name
	 */
	public $bucket;
	
	/**
	 * @var string file key
	 */
	public $key;

	public function __construct($bucket, $key)
	{
		$this->bucket = $bucket;
		$this->key = $key;
	}
}

/**
 * 二元操作路径
 */
class Qiniu_RS_EntryPathPair
{
	/**
	 * @var Qiniu_RS_EntryPath
	 */
	public $src;
	
	/**
	 * @var Qiniu_RS_EntryPath
	 */
	public $dest;

	public function __construct($src, $dest)
	{
		$this->src = $src;
		$this->dest = $dest;
	}
}

/**
 * 查看单个文件的信息
 * @param Qiniu_MacHttpClient $self
 * @param string $bucket
 * @param string $key
 * @return array($statRet, Qiniu_ResponseError $error)
 */
function Qiniu_RS_Stat($self, $bucket, $key) {}

/**
 * 删除单个文件
 * @param Qiniu_MacHttpClient $self
 * @param string $bucket
 * @param string $key
 * @return Qiniu_ResponseError $error
 */
function Qiniu_RS_Delete($self, $bucket, $key) {}

/**
 * 移动文件
 * @param Qiniu_MacHttpClient $self
 * @param string $bucketSrc
 * @param string $keySrc
 * @param string $bucketDest
 * @param string $keyDest
 * @return  Qiniu_ResponseError $error
 */
function Qiniu_RS_Move($self, $bucketSrc, $keySrc, $bucketDest, $keyDest) {}

/**
 * 文件复制
 * @param Qiniu_MacHttpClient $self
 * @param string $bucketSrc
 * @param string $keySrc
 * @param string $bucketDest
 * @param string $keyDest
 * @return Qiniu_ResponseError $error
 */
function Qiniu_RS_Copy($self, $bucketSrc, $keySrc, $bucketDest, $keyDest) {}

function Qiniu_RS_Batch($self, $ops) {}

/**
 * 批量文件查看
 * @param Qiniu_MacHttpClient $self
 * @param  Array Qiniu_RS_EntryPath $entryPaths
 * @return array($statRet, Qiniu_ResponseError $err)
 */
function Qiniu_RS_BatchStat($self, $entryPaths) {}

/**
 * 批量文件查看
 * @param Qiniu_MacHttpClient $self
 * @param  Array Qiniu_RS_EntryPath $entryPaths
 * @return array($statRet, Qiniu_ResponseError $err)
 */
function Qiniu_RS_BatchDelete($self, $entryPaths) {}

/**
 * 批量文件查看
 * @param Qiniu_MacHttpClient $self
 * @param  Array Qiniu_RS_EntryPathPair $entryPairs
 * @return array($statRet, Qiniu_ResponseError $err)
 */
function Qiniu_RS_BatchMove($self, $entryPairs) {}

/**
 * 批量文件查看
 * @param Qiniu_MacHttpClient $self
 * @param  Array Qiniu_RS_EntryPathPair $entryPairs
 * @return array($statRet, Qiniu_ResponseError $err)
 */
function Qiniu_RS_BatchCopy($self, $entryPairs) {}

```

范围：仅在服务端使用


## 上传/下载授权凭证（uptoken/dntoken）

```
{php}

class Qiniu_RS_PutPolicy
{

	/**
	 * 一般指文件要上传到的目标存储空间（Bucket）。
	 * 若为”Bucket”，表示限定只能传到该Bucket（仅限于新增文件）。
	 * 若为”Bucket:Key”，表示限定特定的文件，可修改该文件。
	 */
	public $Scope;
	
	/**
	 *  文件上传成功后，Qiniu-Cloud-Server 向 App-Server 发送POST请求的URL.
	 *  必须是公网上可以正常进行POST请求并能响应 HTTP Status 200 OK 的有效 URL
	 */
	public $CallbackUrl;
	
	/**
	 * 文件上传成功后，Qiniu-Cloud-Server 向 App-Server 发送POST请求的数据。
	 * 支持 魔法变量 和 自定义变量，不可与 returnBody 同时使用。
	 */
	public $CallbackBody;
	
	/**
	 * 设置用于浏览器端文件上传成功后，浏览器执行301跳转的URL，一般为 HTML Form 上传时使用。
	 * 文件上传成功后会跳转到 returnUrl?query_string, query_string 会包含 returnBody 内容。
	 * returnUrl 不可与 callbackUrl 同时使用。
	 */
	public $ReturnUrl;
	
	/**
	 *  文件上传成功后，自定义从 Qiniu-Cloud-Server 最终返回給终端 App-Client 的数据。
	 *  支持 魔法变量，不可与 callbackBody 同时使用。
	 */
	public $ReturnBody;
	
	/**
	 * 指定文件（图片/音频/视频）上传成功后异步地执行指定的预转操作。
	 * 每个预转指令是一个API规格字符串，多个预转指令可以使用分号“;”隔开。
	 */
	public $AsyncOps;
	
	/**
	 * 给上传的文件添加唯一属主标识，特殊场景下非常有用，比如根据终端用户标识给图片或视频打水印
	 */
	public $EndUser;
	
	/**
	 * 定义 uploadToken 的失效时间，Unix时间戳，精确到秒，缺省为 3600 秒
	 */
	public $Expires;

	
	/**
	 * 构造函数
	 */
	public function __construct($scope)
	{
		$this->Scope = $scope;
	}

	/**
	 * 生成upload token
	 */
	public function Token($mac) {}
	
}

class Qiniu_RS_GetPolicy
{

	/**
	 * 可选， 默认3600秒
	 */
	public $Expires;

	/**
	 * 对请求url 进行前面
	 */
	public function MakeRequest($baseUrl, $mac) {}
}

/**
 * 构造请求url
 */
function Qiniu_RS_MakeBaseUrl($domain, $key) {}
```

范围：仅在服务端使用


## 存储高级API（rsf）

```
{php}
/**
 * 1. 首次请求 marker = ""
 * 2. 无论 err 值如何，均应该先看 items 是否有内容
 * 3. 如果后续没有更多数据，err 返回 EOF，markerOut 返回 ""（但不通过该特征来判断是否结束）
 * @param $self Qiniu_MacHttpClient
 * @param $bucket string 
 * Bucket name.
 * @param $prefix string
 * 要list文件的前缀
 * @param $marker string
 * Fetch 定位符.
 * @param $limit int
 * Fetch返回结果条目数量限制
 *
 */
function Qiniu_RSF_ListPrefix(
	$self, $bucket, $prefix = '', $marker = '', $limit = 0) {}
```


## 上传（io）

```
{php}

class Qiniu_PutExtra
{
	/**
	 *  用户自定义参数，key必须以 "x:" 开头
	 */
	public $Params = null;
	
	/**
	 * 文件的媒体类型
	 */
	public $MimeType = null;
	
	/**
	 * crc32 值
	 */
	public $Crc32 = 0;
	
	/**
	 * 是否对上传内容进行 crc32 检验
	 * CheckCrc == 0: 表示不进行 crc32 校验
	 * CheckCrc == 1: 对于 Put 等同于 CheckCrc = 2；对于 PutFile 会自动计算 crc32 值
	 * CheckCrc == 2: 表示进行 crc32 校验，且 crc32 值就是上面的 Crc32 变量
	 */
	public $CheckCrc = 0;
}

function Qiniu_Put($upToken, $key, $body, $putExtra) {}

function Qiniu_PutFile($upToken, $key, $localFile, $putExtra) {}

```

范围：客户端和服务端


## 断点续上传（resumable io）

```

{php}

class Qiniu_Rio_PutExtra
{
	/**
	 * @var string 必选 （未来会没有这个字段）。
	 */
	public $Bucket = null;		
	
	/**
	 * @var array  用户自定义参数，key必须以 "x:" 开头
	 */
	public $Params = null;
	
	/**
	 * @var string
	 */
	public $MimeType = null;
	
	/**
	 * @var int 可选。每次上传的Chunk大小
	 */
	public $ChunkSize = 0;		
	
	/**
	 * @var int 可选。尝试次数
	 */
	public $TryTimes = 0;	

	/**
	 * @var []BlkputRet 可选。上传进度
	 */
	public $Progresses = null;
	
	/**
	 * @var func(blkIdx int, blkSize int, ret *BlkputRet) 进度通知
	 */
	public $Notify = null;		

	/**
	 * @var func(blkIdx int, blkSize int, err error) 错误通知
	 */
	public $NotifyErr = null;

	public function __construct($bucket = null) {
		$this->Bucket = $bucket;
	}
}


function Qiniu_Rio_Mkblock ($self, $host, $reader, $size) {} 
function Qiniu_Rio_Mkfile($self, $host, $key, $fsize, $extra) {}

class Qiniu_Rio_UploadClient
{
	public $uptoken;

	public function __construct($uptoken)
	{
		$this->uptoken = $uptoken;
	}

	public function RoundTrip ($req) {}
		
}


/**
 * @param string $upToken
 * @param string $key
 * @param mixed $body
 * @param int $fsize
 * @param  Qiniu_Rio_PutExtra $putExtra
 * @return array($putRet, $err)
 */
function Qiniu_Rio_Put($upToken, $key, $body, $fsize, $putExtra) {}

/**
 * @param string $upToken
 * @param string $key
 * @param string $localFile
 * @param Qiniu_Rio_PutExtra $putExtra
 * @return array($putRet, $err)
 */
function Qiniu_Rio_PutFile($upToken, $key, $localFile, $putExtra) {}

```

范围：客户端和服务端


## 数据处理API（fop）

```
{php}

class Qiniu_ImageView {
	/**
	 * @var int 缩略模式
	 */
	public $Mode;
	
	/**
	 * @var int 不设置时，不限定宽度。
	 */
    public $Width;
    
    /**
     * @var int 不设置时，不限定宽度。
     */
    public $Height;
    
    /**
     * @var int 图片质量： 1-100
     */
    public $Quality;
    
    /**
     * @var string 图片输出格式 如 png, jpg 等
     */
    public $Format;

    /**
     * @param string $url
     * 图片的url
     * @return string
     */
    public function MakeRequest($url) {}
}

class Qiniu_Exif {
	/**
	 * @param string $url
	 * 图片的url
	 * @return string
	 */
	public function MakeRequest($url) {}

}

class Qiniu_ImageInfo {

	/**
	 * @param string $url
	 * 图片的url
	 * @return string
	 */
	public function MakeRequest($url) {}
}
```

范围：客户端和服务端

