# Qiniu Cloud Storage SDK Specification

[![Qiniu Logo](http://qiniutek.com/images/logo-2.png)](http://qiniu.com/)


- 命名风格：按照 [http://nodeguide.com/style.html](http://nodeguide.com/style.html)
- callback：  sdk中出现的`onret`都是`function(err, resp){}`形式
- 以Key为对象进行大部分操作，支持stream上传下载。


## 服务端配置（conf）

```{js}
USER_AGENT = ''// 请求的 User-Agent 值，比如 'qiniu nodejs-sdk v7.0.0'

UP_HOST = 'http://up.qiniu.com';
RS_HOST = 'http://rs.qiniu.com';
RSF_HOST = 'http://rsf.qiniu.com';

ACCESS_KEY = '<PLEASE APPLY YOUR ACCESS KEY>';
SECRET_KEY = '<DONT SEND YOUR SECRET KEY TO ANYONE>';

conf = {
	'accessKey': accessKey,
	'secretKey': secretKey
}
exports.SetConf(conf)
```

范围：服务端和客户端共用

## bucket

```{js}
exports.Bucket = Bucket

conf = {
	...
	'bucketName': bucketname,
	'dnHost': dnHost,  // 没有指定则为 http://<bucketname>.qiniudn.com
	'public': true || false
}
Bucket = function(conf) {}

// 获取key，keyname可为null
Bucket.prototype.key = function(keyname) {}
// 列出 prefix 前缀的文件
Bucket.prototype.listPrefix = function(prefix, marker, limit, onret) {}
```

范围：服务端和客户端共用

## 存储API（rs）

```{js}
Key.prototype.stat = function(onret) {}
Key.prototype.remove = function(onret) {}
Key.prototype.move = function(dstKey, onret) {}
Key.prototype.copy = function(dstKey, onret) {}

// conf 在操作非default accesskey时候用
function Batch(conf) {}

Batch.prototype.stat = function([key, ...], onret) {}
Batch.prototype.copy = function([[src, dst],...], onret) {}
Batch.prototype.move = function([[src, dst],...], onret) {}
Batch.prototype.remove = function([key, ...], onret) {}
```

范围：仅在服务端使用


## 上传/下载授权凭证（uptoken/dntoken）

```{js}
putPolicy = {
	'callbackUrl': callbackUrl,
	'callbackBody': callbackBody,
	'returnUrl': returnUrl,
	'returnBody': returnBody,
	'asyncOps': asyncOps,
	'endUser': endUser,
	'expires': expires
}

// bucket
Bucket.prototype.uptoken = function(putPolicy)
// bucket:key
Key.prototype.uptoken = function(putPolicy)
```

范围：仅在服务端使用


## 上传（io）

```{js}
extra = {
	'params': {...},
	'mimeType': 'image/webp',
	'crc32': 'crc32 code',
	'checkCrc': 0 || 1 || 2
		// checkCrc == 0: 表示不进行 crc32 校验
		// checkCrc == 1: 对于 Put 等同于 checkCrc = 2；对于 PutFile 会自动计算 crc32 值
		// checkCrc == 2: 表示进行 crc32 校验，且 crc32 值就是上面的 crc32 变量
}

// 可以指定2、3、4个参数
// 从内存中上传文件
Key.prototype.put = function(body, extra, uptoken, onret) {} 
// 根据文件名上传文件
Key.prototype.putFile = function(loadFile, extra, uptoken, onret) {}
// 根据文件名上传文件，不指定key


Key.prototype.setToken = function(token) {}
Key.prototype.setExtra = function(extra) {}

read = createReadStream("xxx");
read.pipe(key)

// download, auto public or private
Key.prototype.downloadUrl = function(expires) {}
```

范围：客户端和服务端

## 数据处理API（fop）

```{js}
// imageInfo
Key.prototype.imageInfoUrl = function() {}
Key.prototype.imageInfoCall = function(onret) {}

// exif
Key.prototype.exifUrl = function() {}
Key.prototype.exifCall = function(onret) {}

// qrcode
Key.prototype.qrcodeUrl = function() {}

// imageView
imageView = {
	'mode': mode,		// 缩略模式
	'width': width,		// Width = 0 表示不限定宽度
	'height': height,	// Height = 0 表示不限定高度
	'quality': quality,	// 质量, 1-100
	'format': format	// 输出格式，如jpg, gif, png, tif等等
}

Key.prototype.imageViewUrl = function(imageView) {}

// imageMogr
imageMogr = {
	...
}

Key.prototype.imageMogrUrl = function(imageMogr) {}
```
