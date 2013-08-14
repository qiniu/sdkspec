# Qiniu Cloud Storage C# SDK Specification

[![Qiniu Logo](http://qiniutek.com/images/logo-2.png)](http://qiniu.com/)

## 服务端配置（conf）

服务端app.config 文件配置节点
``` xml
<configuration>
 	<appSettings> 
        <add key = "USER_AGENT" value="qiniu csharp-sdk version" />
		<add key = "ACCESS_KEY" value="Please apply your access key" />
		<add key = "SECRET_KEY" value="Dont send your secret key to anyone" />
		<add key = "RS_HOST" value="http://rs.Qbox.me" />
		<add key = "UP_HOST" value="http://up.qiniu.com" />
		<add key = "RSF_HOST" value="http://rsf.Qbox.me" />
	</appSettings> 
</configuration>
```

``` c#
namespace Qiniu.Conf
{
	public class Config
	{
		public static string USER_AGENT = "qiniu csharp-sdk v6.0.0";
		#region 帐户信息
		/// <summary>
		/// 七牛提供的公钥，用于识别用户
		/// </summary>
		public static string ACCESS_KEY = "<Please apply your access key>";
		/// <summary>
		/// 七牛提供的秘钥，不要在客户端初始化该变量
		/// </summary>
		public static string SECRET_KEY = "<Dont send your secret key to anyone>";
		#endregion
		#region 七牛服务器地址
		/// <summary>
		/// 七牛资源管理服务器地址
		/// </summary>
		public static string RS_HOST;
		/// <summary>
		/// 七牛资源上传服务器地址.
		/// </summary>
		public static string UP_HOST;
		/// <summary>
		/// 七牛资源列表服务器地址.
		/// </summary>
		public static string RSF_HOST;
		#endregion
		/// <summary>
		/// 七牛SDK对所有的字节编码采用utf-8形式 .
		/// </summary>
		public static Encoding Encoding = Encoding.UTF8;

		/// <summary>
		/// 初始化七牛帐户、请求地址等信息，不应在客户端调用。
		/// </summary>
		public static void Init ();
	}
}
```

范围：服务端和客户端共用


## 签名认证（auth/digest）

``` c#
namespace Qiniu.Auth.digest
{
	/// <summary>
	/// 七牛消息认证(Message Authentication)
	/// </summary>
	public class Mac
	{
		/// <summary>
		/// Gets or sets the access key.
		/// </summary>
		public string AccessKey {
			get; set;
		}

		/// <summary>
		/// Gets the secret key.
		/// </summary>
		public byte[] SecretKey {
			get;
		}

		public Mac ();//直接从Config中读取帐户信息
		public Mac (string access, byte[] secretKey);
		public string Sign (byte[] b); 
		public string SignWithData (byte[] data);
	}
}
```

范围：仅在服务端使用


## 存储API（rs）

``` c#
namespace Qiniu.RS
{
	public class Entry 
	{
		/// <summary>
		/// 文件的Hash值
		/// </summary>
		public string Hash { get; private set; }

		/// <summary>
		/// 文件的大小(单位: 字节)
		/// </summary>
		public long Fsize { get; private set; }

		/// <summary>
		/// 文件上传到七牛云的时间(Unix时间戳)
		/// </summary>
		public long PutTime { get; private set; }

		/// <summary>
		/// 文件的媒体类型，比如"image/gif"
		/// </summary>
		public string MimeType { get; private set; }

		/// <summary>
		/// Gets the customer.
		/// </summary>
		public string Customer { get; private set; }
	}

	/// <summary>
	/// 资源路径 bucket+   ":"+ key
	/// </summary>
	public class EntryPath
	{
		/// <summary>
		/// 七年云存储空间名
		/// </summary>
		public string Bucket {get;}

		/// <summary>
		/// 文件key
		/// </summary>
		public string Key {get;}

		public EntryPath (string bucket, string key);
	}
	/// <summary>
	/// 二元操作路径
	/// </summary>
	public class EntryPathPair
	{
		EntryPath src;
		EntryPath dest;
		/// <summary>
		/// 二元操作路径构造函数
		/// </summary>
		/// <param name="bucketSrc">源空间名称</param>
		/// <param name="keySrc">源文件key</param>
		/// <param name="bucketDest">目标空间名称</param>
		/// <param name="keyDest">目文件key</param>
		public EntryPathPair (string bucketSrc, string keySrc, string bucketDest, string keyDest);

		/// <summary>
		/// 二元操作路径构造函数
		/// </summary>
		/// <param name="bucket">源空间名称，目标空间名称</param>
		/// <param name="keySrc">源文件key</param>
		/// <param name="keyDest">目文件key</param>
		public EntryPathPair (string bucket, string keySrc, string keyDest);
	}

	/// <summary>
	/// 批操作返回结果
	/// </summary>
	public class BatchRetItem
	{
		public int code;
		public BatchRetData data;
	}

	/// <summary>
	/// 批操作返回结果
	/// </summary>
	public class BatchRetData
	{
		/// <summary>
		/// 文件大小.
		/// </summary>
		public Int64 FSize {get; set;}

		/// <summary>
		/// 修改时间.
		/// </summary>
		public Int64 PutTime {get; set;}

		/// <summary>
		/// 文件名.
		/// </summary>
		public string Key {get; set;}

		/// <summary>
		/// Gets a value indicating whether this instance hash.
		/// </summary>
		public string Hash {get; set ;}

		/// <summary>
		/// Gets the MIME.
		/// </summary>
		public string Mime {get; set;}

		/// <summary>
		/// Gets the EndUser
		/// </summary>
		public string EndUser {get; set;}

		/// <summary>
		/// Error 
		/// </summary>
		public string Error {get; set; }
	}

	/// <summary>
	/// 资源存储客户端，提供对文件的查看（stat），移动(move)，复制（copy）,删除（delete）操作
	/// 以及与这些操作对应的批量操作
	/// </summary>
	public class RSClient 
	{
		private static string[] OPS = new string[] { "stat", "move", "copy", "delet" };

		public RSClient (Mac mac=null);

		#region 单文件操作
		/// <summary>
		/// 查看文件
		/// </summary>
		public Entry Stat (EntryPath scope);

		/// <summary>
		/// 删除文件
		/// </summary>
		public CallRet Delete (EntryPath scope);

		/// <summary>
		/// 移动文件
		/// </summary>
		public CallRet Move (EntryPathPair pathPair);

		/// <summary>
		/// 复制
		/// </summary>
		public CallRet Copy (EntryPathPair pathPair);
		#endregion

		
		#region 批文件操作
		/// <summary>
		/// 批操作：文件查看
		/// </summary>
		public List<BatchRetItem> BatchStat (EntryPath[] keys);
		
		/// <summary>
		/// 批操作：文件移动
		/// </summary>
		public CallRet BatchMove (EntryPathPair[] entryPathPairs);

		/// <summary>
		/// 批操作：文件复制
		/// </summary>
		public CallRet BatchCopy (EntryPathPair[] entryPathPari);

		/// <summary>
		/// 批操作：删除
		/// </summary>
		public CallRet BatchDelete (EntryPath[] keys);
		#endregion
	}
}
```

范围：仅在服务端使用


## 上传/下载授权凭证（uptoken/dntoken）

``` c#
namespace Qiniu.RS
{
	/// <summary>
	/// 上传凭证
	/// </summary>
	public class PutPolicy
	{
		/// <summary>
		/// 一般指文件要上传到的目标存储空间（Bucket）。若为”Bucket”，表示限定只能传到该Bucket（仅限于新增文件）；若为”Bucket:Key”，表示限定特定的文件，可修改该文件。
		/// </summary>
		public string Scope {get; set; }

		/// <summary>
		/// 文件上传成功后，Qiniu-Cloud-Server 向 App-Server 发送POST请求的URL，必须是公网上可以正常进行POST请求并能响应 HTTP Status 200 OK 的有效 URL
		/// </summary>
		public string CallBackUrl {get; set; }

		/// <summary>
		/// 文件上传成功后，Qiniu-Cloud-Server 向 App-Server 发送POST请求的数据。支持 魔法变量 和 自定义变量，不可与 returnBody 同时使用。
		/// </summary>
		public string CallBackBody  {get; set; }

		/// <summary>
		/// 设置用于浏览器端文件上传成功后，浏览器执行301跳转的URL，一般为 HTML Form 上传时使用。文件上传成功后会跳转到 returnUrl?query_string, query_string 会包含 returnBody 内容。returnUrl 不可与 callbackUrl 同时使用
		/// </summary>
		public string ReturnUrl {get; set; }

		/// <summary>
		/// 文件上传成功后，自定义从 Qiniu-Cloud-Server 最终返回給终端 App-Client 的数据。支持 魔法变量，不可与 callbackBody 同时使用。
		/// </summary>    
		public string ReturnBody {get; set; }

		/// <summary>
		/// 指定文件（图片/音频/视频）上传成功后异步地执行指定的预转操作。每个预转指令是一个API规格字符串，多个预转指令可以使用分号“;”隔开
		/// </summary>
		public string AsyncOps {get; set; }

		/// <summary>
		/// 给上传的文件添加唯一属主标识，特殊场景下非常有用，比如根据终端用户标识给图片或视频打水印
		/// </summary>
		public string EndUser {get; set; }

		/// <summary>
		/// 定义 uploadToken 的失效时间，Unix时间戳，精确到秒，缺省为 3600 秒
		/// </summary>
		[JsonProperty("deadline")]
		public string Expires {get; set; }

		/// <summary>
		/// 构造函数  
		/// </summary>
		public PutPolicy (string scope, UInt32 expires=3600);

		/// <summary>
		/// 生成上传Token
		/// </summary>
		public string Token (Mac mac=null);
	}

	/// <summary>
	/// 下载凭证
	/// </summary>
	public class GetPolicy
	{
		/// <summary>
		/// 对请求进行签名
		/// </summary>
		public static string MakeRequest (string baseUrl, UInt32 expires = 3600, Mac mac = null);
		
		/// <summary>
		/// 构造请求URL
		/// </summary>
		public static string MakeBaseUrl (string domain, string key);
	}
}
```

范围：仅在服务端使用


## 存储高级API（rsf）

``` c#
namespace Qiniu.RSF
{	
	/// <summary>
	/// RS Fetch 
	/// </summary>
	public class RSFClient : QiniuAuthClient
	{        
		//单次fetch获取结果数目最大限制
		private const int MAX_LIMIT = 1000;
		//失败重试次数
		private const int RETRY_TIME = 3;

		/// <summary>
		/// bucket name
		/// </summary>
		public string BucketName { get; private set; }

		/// <summary>
		/// Fetch返回结果条目数量限制
		/// </summary>
		public int Limit {get ; set ; } 

		/// <summary>
		/// 文件前缀
		/// </summary>
		public string Prefix {get ; set ; }

		/// <summary>
		/// Fetch 定位符.
		/// </summary>
		public string Marker  {get ; set ; }

		/// <summary>
		/// RS Fetch Client
		/// </summary>
		public RSFClient (string bucketName);

		/// <summary>
		/// 获取资源列表
		/// </summary>
		public DumpRet ListPrefix (string bucketName, string prefix="", string markerIn="", int limit = MAX_LIMIT);
	}
}
```

范围：仅在服务端使用


## 上传（io）

``` c#
namespace Qiniu.IO
{

	/// <summary>
	/// CRC检查类型
	/// </summary>
	public enum CheckCrcType
	{
		/// <summary>
		/// default
		/// </summary>
		DEFAULT_CHECK = -1,
		/// <summary>
		/// 表示不进行 crc32 校验
		/// </summary>
		NO_CHECK = 0,
		/// <summary>
		///对于 Put 等同于 CheckCrc = 2；对于 PutFile 会自动计算 crc32 值
		/// </summary>
		CHECK_AUTO = 1,
		/// <summary>
		/// 表示进行 crc32 校验，且 crc32 值就是PutExtra:Crc32
		/// </summary>
		CHECK = 2
	}

	public class PutExtra
	{
		/// <summary>
		/// 用户自定义参数，key必须以 "x:" 开头
		/// </summary>
		public Dictionary<string, string> Params{ get; set; }

		/// <summary>
		/// 文件的媒体类型
		/// </summary>
		public string MimeType { get; set; }

		/// <summary>
		/// crc32值 
		/// </summary>
		public Int32 Crc32 { get; set; }
		public CheckCrcType CheckCrc { get; set; }
		public string Scope { get; set; }

		public PutExtra ();
		public PutExtra (string bucket, string mimeType);
	}

	public class PutRet 
	{
		/// <summary>
		/// 如果 uptoken 没有指定 ReturnBody，那么返回值是标准的 PutRet 结构
		/// </summary>
		public string Hash { get; private set; }

		/// <summary>
		/// 如果传入的 key == UNDEFINED_KEY，则服务端返回 key
		/// </summary>
		public string key { get; private set; }
	}
	/// <summary>
    /// 上传客户端
    /// </summary>
    public class IOClient
    {
	    private const string UNDEFINED_KEY = "?"
        /// <summary>
        /// 无论成功或失败，上传结束时触发的事件
        /// </summary>
        public event EventHandler<PutRet> PutFinished;

        /// <summary>
        /// 上传文件
        /// </summary>
        public PutRet PutFile(string upToken, string key, string localFile, PutExtra extra);

        /// <summary>
		/// Puts the file without key.
		/// </summary>
		public PutRet PutFileWithoutKey(string upToken,string localFile,PutExtra extra);

        /// <summary>
        /// 上传流 
        /// </summary>
        public PutRet Put(string upToken, string key, System.IO.Stream putStream, PutExtra extra);
    }
}
```

范围：客户端和服务端


## 断点续上传（resumable io）

``` c#
namespace Qiniu.IO.Resumable
{
	/// <summary>
	/// Block上传成功事件参数
	/// </summary>
	public class PutNotifyEvent:EventArgs
	{
		public int BlkIdx {get;}

		public int BlkSize {get;}

		public BlkputRet Ret {get;}

		public PutNotifyEvent (int blkIdx, int blkSize, BlkputRet ret);
	}

	/// <summary>
	/// 上传错误事件参数
	/// </summary>
	public class PutNotifyErrorEvent:EventArgs
	{
		public int BlkIdx {get;}

		public int BlkSize {get;}

		public string Error {get;}

		public PutNotifyErrorEvent (int blkIdx, int blkSize, string error);
	}

	/// <summary>
	/// 断点续上传参数 
	/// </summary>
	public class ResumablePutExtra
	{
		public string CallbackParams;
		public string Bucket;
		public string CustomMeta;
		public string MimeType;
		public int chunkSize;
		public int tryTimes;
		public BlkputRet[] Progresses;
		public event EventHandler<PutNotifyEvent> Notify;
		public event EventHandler<PutNotifyErrorEvent> NotifyErr;
	}

	/// <summary>
	/// 块上传结果 
	/// </summary>
	public class BlkputRet
	{
		public string ctx;
		public string checkSum;
		public UInt32 crc32;
		public UInt32 offset;
	}

	/// <summary>
	/// 异步并行断点上传类
	/// </summary>
	public class ResumablePut
	{
		private const string UNDEFINED_KEY = "?"
		/// <summary>
		/// 上传完成事件
		/// </summary>
		public event EventHandler<CallRet> PutFinished;

		/// <summary>
		/// 进度提示事件
		/// </summary>
		public event Action<float> Progress;

		/// <summary>
		/// 上传设置
		/// </summary>
		public Settings PutSetting {get; set;}

		/// <summary>
		/// PutExtra
		/// </summary>
		public ResumablePutExtra Extra {get; set;}

		/// <summary>
		/// 构造函数
		/// </summary>
		public ResumablePut (Settings putSetting, ResumablePutExtra extra);

		/// <summary>
		/// 上传文件
		/// </summary>
		public void PutFile (string upToken, string localFile, string key);
	}
}
```

范围：客户端和服务端


## 数据处理API（fop）

``` c#
namespace Qiniu.FileOP{
	public class ImageView
	{
		/// <summary>
		/// 缩略模式
		/// </summary>
		public int Mode { get; set; }

		/// <summary>
		/// Width = 0 表示不限定宽度
		/// </summary>
		public int Width { get; set; }

		/// <summary>
		/// Height = 0 表示不限定高度
		/// </summary>
		public int Height { get; set; }

		/// <summary>
		///质量, 1-100
		/// </summary>
		public int Quality { get; set; }

		/// <summary>
		/// 输出格式，如jpg, gif, png, tif等等
		/// </summary>
		public string Format { get; set; }

		/// <summary>
		/// Makes the request.
		/// </summary>
		public string MakeRequest (string url);
	}

	/// <summary>
	///高级图像处理接口（第二版）（缩略、裁剪、旋转、转化） 
	/// </summary>
	public class ImageMogrify
	{
		public bool AutoOrient { get; set; }

		public string Thumbnail { get; set; }

		public string Gravity { get; set; }

		public string Crop { get; set; }

		public int Quality { get; set; }

		public int Rotate { get; set; }

		public string Format { get; set; }

		public string MakeRequest (string url);
	}
	

	/// <summary>
	/// 获取图片基本信息，图片基本信息包括图片格式,图片大小，色彩模式
	/// </summary>
	public class ImageInfoRet 
	{
		/// <summary>
		/// "png", "jpeg", "gif", "bmp", etc.
		/// </summary>
		public int Width { get; private set; }

		/// <summary>
		/// 图片高度 
		/// </summary>
		public int Height { get; private set; }

		/// <summary>
		/// 图片宽度
		/// </summary>
		public string Format { get; private set; }

		/// <summary>
		/// "palette16", "ycbcr", etc.
		/// </summary>
		public string ColorModel { get; private set; }
	}

	public static class ImageInfo
	{
		public static string MakeRequest (string url);

		public static ImageInfoRet Call (string url);
	}

	public class ExifValType
	{
		public string val { get; set; }
		public int type { get; set; }
	}
	public class ExifRet : CallRet
	{
		public ExifValType this [string key] {get;}
	}
}
```

范围：客户端和服务端

