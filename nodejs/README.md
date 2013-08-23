# Qiniu Cloud Storage SDK Specification

[![Qiniu Logo](http://qiniutek.com/images/logo-2.png)](http://qiniu.com/)


- 命名风格：按照 [http://nodeguide.com/style.html](http://nodeguide.com/style.html)
- callback：  sdk中出现的`onret`都是`function(err, resp){}`形式


## 服务端配置（conf）

```{js}
exports.USER_AGENT = ''// 请求的 User-Agent 值，比如 'qiniu nodejs-sdk v6.0.0'

exports.UP_HOST = 'http://up.qbox.me';
exports.RS_HOST = 'http://rs.qbox.me';
exports.RSF_HOST = 'http://rsf.qbox.me';

exports.ACCESS_KEY = '<PLEASE APPLY YOUR ACCESS KEY>';
exports.SECRET_KEY = '<DONT SEND YOUR SECRET KEY TO ANYONE>';
```

范围：服务端和客户端共用


## 签名认证（auth/digest）

```{js}
function Mac(accesskey, secretkey) {
    this.accesskey = accesskey || conf.ACCESS_KEY;
    this.secretkey = secretkey || conf.SECRET_KEY;
}
```

范围：仅在服务端使用


## 存储API（rs）

```{js}
exports.Client = Client;
exports.Entry = Entry;
exports.EntryPath = EntryPath;
exports.EntryPathPair = EntryPathPair;
exports.BatchItemRet = BatchItemRet;
exports.BatchStatItemRet = BatchStatItemRet;

exports.PutPolicy = PutPolicy;
exports.GetPolicy = GetPolicy;
exports.makeBaseUrl = makeBaseUrl;

function Client(client) {
  this.client = client || null;
}

// 查看文件信息
Client.prototype.stat = function(bucket, key, onret) {}
// 删除文件
Client.prototype.remove = function(bucket, key, onret) {}
// 移动文件
Client.prototype.move = function(bucketSrc, keySrc, bucketDest, keyDest, onret) {}
// 复制文件
Client.prototype.copy = function(bucketSrc, keySrc, bucketDest, keyDest, onret) {}

// batch

function EntryPath(bucket, key) {
  this.bucket = bucket || null;
  this.key = key || null;
}

function EntryPathPair(src, dest) {
  this.src = src || null; 
  this.dest = dest || null;
}

// 批量查看文件, entries 为 EntryPath 的数组
Client.prototype.batchStat = function(entries, onret) {}
// 批量删除文件, entries 为 EntryPath 的数组
Client.prototype.batchDelete = function(entries, onret) {}
// 批量移动文件, entries 为 EntryPathPair 的数组
Client.prototype.batchMove = function(entries, onret) {}
// 批量复制文件, entries 为 EntryPathPair 的数组
Client.prototype.batchCopy = function(entries, onret) {}
```

范围：仅在服务端使用


## 上传/下载授权凭证（uptoken/dntoken）

```{js}

function PutPolicy(scope, callbackUrl, callbackBody, returnUrl, returnBody,
                  asyncOps, endUser, expires) {
  this.scope = scope || null;
  this.callbackUrl = callbackUrl || null;
  this.callbackBody = callbackBody || null;
  this.returnUrl = returnUrl || null;
  this.returnBody = returnBody || null;
  this.asyncOps = asyncOps || null;
  this.endUser = endUser || null;
  this.expires = expires || 3600;
}

// 生成上传的 token
PutPolicy.prototype.token = function(mac) {}

function GetPolicy(expires) {
  this.expires = expires || 3600;
}

// 生成私有下载的 url
GetPolicy.prototype.makeRequest = function(baseUrl, mac) {}

function makeBaseUrl(domain, key) {}
```

范围：仅在服务端使用


## 存储高级API（rsf）

```{js}
// 列出 prefix 前缀的文件
exports.listPrefix = function(bucket, prefix, marker, limit, onret) {}
```

范围：仅在服务端使用


## 上传（io）

```{js}
exports.PutExtra = PutExtra;
exports.put = put;
exports.putWithoutKey = putWithoutKey;
exports.putFile = putFile;
exports.putFileWithoutKey = putFileWithoutKey;

function PutExtra(params, mimeType, crc32, checkCrc) {
  this.paras = params || {};
  this.mimeType = mimeType || null;
  this.crc32 = crc32 || null;
  this.checkCrc = checkCrc || 0;
		// checkCrc == 0: 表示不进行 crc32 校验
		// checkCrc == 1: 对于 Put 等同于 checkCrc = 2；对于 PutFile 会自动计算 crc32 值
		// checkCrc == 2: 表示进行 crc32 校验，且 crc32 值就是上面的 crc32 变量
}

// 从内存中上传文件
function put(uptoken, key, body, extra, onret) {}
// 从内存中上传文件，不指定key
function putWithoutKey(uptoken, body, extra, onret) {}
// 根据文件名上传文件
function putFile(uptoken, key, loadFile, extra, onret) {}
// 根据文件名上传文件，不指定key
function putFileWithoutKey(uptoken, loadFile, extra, onret) {}
```

范围：客户端和服务端

## 数据处理API（fop）

```{js}
exports.ImageView = ImageView;
exports.ImageInfo = ImageInfo;
exports.Exif = Exif;

// imageView

function ImageView(mode, width, height, quality, format) {
  this.mode = mode || 1;         // 缩略模式
  this.width = width || 0;       // Width = 0 表示不限定宽度
  this.height = height || 0;     // Height = 0 表示不限定高度
  this.quality = quality || 0;   // 质量, 1-100
  this.format = format || null;  // 输出格式，如jpg, gif, png, tif等等
}

// 生成缩略图的url
ImageView.prototype.makeRequest = function(url) {}

// imageInfo
function ImageInfo() {}

// 生成imageInfo的url
ImageInfo.prototype.makeRequest = function(url) {}

// exif
function Exif() {}

// 生成exif的url
Exif.prototype.makeRequest = function(url) {}
```
