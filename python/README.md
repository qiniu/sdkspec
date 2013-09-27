# Qiniu Cloud Storage SDK Specification

[![Qiniu Logo](http://qiniutek.com/images/logo-2.png)](http://qiniu.com/)

## 语言差异性

- 命名风格：本规范主要按类 Python 风格进行描述。

## 服务端配置（conf）

```{python}
ACCESS_KEY = "<Apply your access key here>"
SECRET_KEY = "<Apply your secret key here>"

RS_HOST = "rs.qbox.me"
RSF_HOST = "rsf.qbox.me"
UP_HOST = "up.qiniu.com"

USER_AGENT = "qiniu python-sdk v%s" % __version__
```

范围：服务端和客户端共用

## 签名认证（auth/digest）

```{python}
import auth.digest

class Mac(object):
	access = None
	secret = None

	def __init__(self, access=None, secret=None):
		self.access = access
		self.secret = secret
```

范围：仅在服务端使用

Bucket

```{python}
conf = {
	"bucket": <bucket_name>,
	"dn_host": <dn_host>,
	"public": True|False
}

class Bucket(Object):
	bucket_name = None
	domain = None
	public = None
	
	def __init__(self, conf=None):
		if isinstance(conf, str):
			self.bucket_name = conf
		if isinstance(conf, dict):
			self.bucket_name = conf["bucket"]
			self.domain = conf"dn_host"]
			self.public = conf["public"]
	
	def key(self, key):
		return <Key>

	def list_prefix(self, prefix, marker, limit):
		return [{
				"Key": <key>,
				"Hash": <hash>,
				"Fsize": <fsize>,
				"PutTime": <putTime>,
				"MimeType": <mimeType>,
				"EndUser": <endUser>
			}
			...
			]

```

## 存储API（rs）

```{python}

class Key(Object):
	bucket_obj = None
	key = None

	def __init__(self, bucket_obj, key):
		self.bucket_obj = bucket_obj
		self.key = key
		
	def stat(self):
		return (<Hash>, <Fsize>, <PutTime>, <MimeType>, <EndUser>)
	def remove(self):
		return err
	def move(self, dst_key):
		"""
		@dst_key: dst_key is also a Key object, for example, Bucket("mybucket").key(["mykey"])
		"""
		return err
	def copy(self, dst_key):
		"""
		@dst_key: dst_key is also a Key object, for example, Bucket("mybucket").key(["mykey"])
		"""
		return err

```

Batch Key operations

```{python}

class Batch_Key(Object):
	bucket_obj = None
	key_list = None

	def __init__(self, bucket_obj, [key1, key2, ...]):
		self.bucket_obj = bucket_obj
		self.key_list = [key1, key2, ...]

	def stat(self):
		return [{
				"data":{"hash":<hash>, "fsize":<fsize>, "mimeType":<mimeType>, "endUser":<EndUser>},
				"error":"err_string",
				"code":<code>
			},
			...
			]
	def remove(self):
		return [{"error":"err_string","code":<code>}...]
	def move(self, dst_batch_key):
		"""
		@dst_batch_key: a Batch_Key object
		"""
		return [{"error":"err_string","code":<code>}...]
	def copy(self, dst_batch_key):
		"""
		@dst_batch_key: a Batch_Key object
		"""
		return [{"error":"err_string","code":<code>}...]
```

范围：仅在服务端使用

## 上传/下载授权凭证（uptoken/dntoken）

```{python}

put_policy = {
	"CallbackUrl": <callbackurl>,
	"CallbackBody: <callbackbody>,
	"ReturnUrl": <returnurl>,
	"ReturnBody": <returnbody>,
	"AsyncOps": <asyncops>,
	"EndUser": <enduser>,
	"Expires": <expires>
}

get_policy = {
	"Expires": <expires>
}

class Bucket(Object):

	... <omit> ...

	def uptoken(self, put_policy):
		""" Up Token of scope <bucket> """
		pass

class Key(Object):

	... <omit> ...
		
	def uptoken(self, put_policy):
		""" uptoken of scope: <bucket>:<key> """
		pass

	def dntoken(self, get_policy): pass

Bucket("foo").uptoken(put_policy)
Bucket("foo").key("bar").uptoken(put_policy)
Bucket("foo").key("bar").dntoken(get_policy)

```
范围：仅在服务端使用


## 上传/下载 (io)

```{python}
extra = {
    'params': {...},
    'mimeType': <mimeType>,
    'crc32': <crc32>,
    'checkCrc': 0 || 1 || 2
        // checkCrc == 0: 表示不进行 crc32 校验
        // checkCrc == 1: 对于 Put 等同于 checkCrc = 2；对于 PutFile 会自动计算 crc32 值
        // checkCrc == 2: 表示进行 crc32 校验，且 crc32 值就是上面的 crc32 变量
}

class Key(Object):

	... <omit> ...

	def put(self, body, extra=None, uptoken=None): pass

	def put_file(self, load_file, extra=None, uptoken=None): pass

	def download_url(self, expires=3600): pass

Bucket("foo").key("bar").put(body, extra, uptoken)
Bucket("foo").key("bar").put(load_file, extra, uptoken)
Bucket("foo").key("bar").download_url(expires)
```

数据处理API（fop）

```{python}

// imageView
imageView = {
    'mode': mode,       // 缩略模式
    'width': width,     // Width = 0 表示不限定宽度
    'height': height,   // Height = 0 表示不限定高度
    'quality': quality, // 质量, 1-100
    'format': format    // 输出格式，如jpg, gif, png, tif等等
}

// imageMogr
imageMogr = {
    ...
}

class Key(Object):

	... <omit> ...

	def image_info_url(self): pass

	def image_info_call(self, image_info_url=None): pass

	def exif_url(self): pass

	def exif_call(self, exif_url=None): pass

	def qrcode_url(self): pass

	def image_view_url(self, image_view): pass

	def image_mogr_url(self, image_mogr): pass

Bucket("foo").key("bar").image_view_url()
Bucket("foo").key("bar").image_mogr_url()
```
