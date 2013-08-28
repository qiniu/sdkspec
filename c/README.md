# Qiniu Cloud Storage SDK Specification

[![Qiniu Logo](http://qiniutek.com/images/logo-2.png)](http://qiniu.com/)

## 语言差异性

- 命名风格：变量命名用小驼峰法，如：`accessKey`
- 名字空间：C中没有命名空间，用前缀区分，如：`Qiniu_`
- 构造函数：通过 NewXXX 函数来表达。

## 服务端配置（conf）

```{c}
#include "conf.h"

const char* QINIU_ACCESS_KEY	= "<Please apply your access key>";
const char* QINIU_SECRET_KEY	= "<Dont send your secret key to anyone>";

const char* QINIU_RS_HOST		= "http://rs.qiniu.com";
const char* QINIU_UP_HOST		= "http://up.qiniu.com";
const char* QINIU_RSF_HOST		= "http://rsf.qiniu.com";

const char* QINIU_USER_AGENT	= "" // 请求的 User-Agent 值，比如 "qiniu c-sdk v6.0.0"
```

范围：服务端和客户端共用

## 签名认证（auth/digest）

```{c}
typedef struct _Qiniu_Mac {
	const char* accessKey;
	const char* secretKey;
} Qiniu_Mac;
```

范围：仅在服务端使用

## 存储API（rs）

```{c}
// "rs.h"

typedef struct _Qiniu_RS_StatRet {
	const char* hash;
	const char* mimeType;
	const char* endUser;
	Qiniu_Int64 fsize;	
	Qiniu_Int64 putTime;
} Qiniu_RS_StatRet;

// 获取文件信息
Qiniu_Error Qiniu_RS_Stat(
	Qiniu_Client* self, Qiniu_RS_StatRet* ret, const char* bucket, const char* key);
// 删除文件
Qiniu_Error Qiniu_RS_Delete(Qiniu_Client* self, const char* bucket, const char* key);
// 移动文件
Qiniu_Error Qiniu_RS_Move(Qiniu_Client* self, 
        const char* tableNameSrc, const char* keySrc, 
        const char* tableNameDest, const char* keyDest);
// 复制文件
Qiniu_Error Qiniu_RS_Copy(Qiniu_Client* self, 
        const char* tableNameSrc, const char* keySrc, 
        const char* tableNameDest, const char* keyDest);

// batch

typedef struct _Qiniu_RS_EntryPath {
    const char* bucket;
    const char* key;
} Qiniu_RS_EntryPath;

typedef struct _Qiniu_RS_EntryPathPair {
    Qiniu_RS_EntryPath src;
    Qiniu_RS_EntryPath dest;
} Qiniu_RS_EntryPathPair;

typedef struct _Qiniu_RS_BatchItemRet {
    const char* error;
    int code;
}Qiniu_RS_BatchItemRet;

typedef struct _Qiniu_RS_BatchStatRet {
    Qiniu_RS_StatRet data;
    const char* error;
    int code;
}Qiniu_RS_BatchStatRet;

// 批量查看文件信息
Qiniu_Error Qiniu_RS_BatchStat(
        Qiniu_Client* self, Qiniu_RS_BatchStatRet* rets,
        Qiniu_RS_EntryPath* entries, Qiniu_ItemCount entryCount);
// 批量删除
Qiniu_Error Qiniu_RS_BatchDelete(
        Qiniu_Client* self, Qiniu_RS_BatchItemRet* rets,
        Qiniu_RS_EntryPath* entries, Qiniu_ItemCount entryCount);
// 批量移动
Qiniu_Error Qiniu_RS_BatchMove(
        Qiniu_Client* self, Qiniu_RS_BatchItemRet* rets,
        Qiniu_RS_EntryPathPair* entryPairs, Qiniu_ItemCount entryCount);
// 批量复制
Qiniu_Error Qiniu_RS_BatchCopy(
        Qiniu_Client* self, Qiniu_RS_BatchItemRet* rets,
        Qiniu_RS_EntryPathPair* entryPairs, Qiniu_ItemCount entryCount);
```

范围：仅在服务端使用

## 上传/下载授权凭证（uptoken/dntoken）

```{c}
// "rs.h"
typedef struct _Qiniu_RS_PutPolicy {
    const char* scope;        // 必选。可以是 bucketName 或者 bucketName:key
    const char* callbackUrl;  // 可选
    const char* callbackBody; // 可选
    const char* returnUrl;    // 可选
    const char* returnBody;   // 可选
    const char* endUser;      // 可选
    const char* asyncOps;     // 可选
    Qiniu_Uint32 expires;     // 可选。默认 3600 秒
} Qiniu_RS_PutPolicy;

// 获取上传token
char* Qiniu_RS_PutPolicy_Token(Qiniu_RS_PutPolicy* policy, Qiniu_Mac* mac);

typedef struct _Qiniu_RS_GetPolicy {
    Qiniu_Uint32 expires;     // 可选。默认 3600 秒
} Qiniu_RS_GetPolicy;

// 获取私有资源url
char* Qiniu_RS_GetPolicy_MakeRequest(Qiniu_RS_GetPolicy* policy, const char* baseUrl, Qiniu_Mac* mac);
// 获取escape转义的url
char* Qiniu_RS_MakeBaseUrl(const char* domain, const char* key);
```

范围：仅在服务端使用

## 存储高级API（rsf）

```{c}
// "rsf.h"

Qiniu_Error Qiniu_RSF_ListPrefix(
	Qiniu_Client* self, Qiniu_RSF_ListItem* ret, const char* bucket, const char* prefix,
	const char* marker, int limit);

type struct _Qiniu_RSF_ListItem {
	const char* key;
	const char* hash;
	const char* mimeType;
	const char* endUser;
	Qiniu_Uint64 fsize;
	Qiniu_Uint64 putTime;
} Qiniu_RSF_ListItem;
```

范围：仅在服务端使用

## 上传（io）

```{c}
// "io.h"

// upload

#ifndef QINIU_UNDEFINED_KEY
#define QINIU_UNDEFINED_KEY NULL
#endif

typedef struct _Qiniu_Io_PutExtraParam {
	const char* key;
	const char* value;
	struct _Qiniu_Io_PutExtraParam* next;
} Qiniu_Io_PutExtraParam;

typedef struct _Qiniu_Io_PutExtra {
	Qiniu_Io_PutExtraParam* params; // 用户自定义参数，key必须以 "x:" 开头
	const char* mimeType;  // 可选
	Qiniu_Uint32 crc32;
	Qiniu_Uint32 checkCrc32;
		// checkCrc32 == 0: 表示不进行 crc32 校验
		// checkCrc32 == 1: 对于 Put 等同于 checkCrc32 = 2；对于 PutFile 会自动计算 crc32 值
		// checkCrc32 == 2: 表示进行 crc32 校验，且 crc32 值就是上面的 crc32 变量
} Qiniu_Io_PutExtra;

typedef struct _Qiniu_Io_PutRet {
	const char* hash; // hash 是对文件内容的hash值
	const char* key;  // 如果传入的 key == UNDEFINED_KEY，则服务端返回 key = hash
} Qiniu_Io_PutRet;

// 从内存中上传文件
Qiniu_Error Qiniu_Io_PutBuffer(
	Qiniu_Client* self, Qiniu_Io_PutRet* ret,
	const char* uptoken, const char* key, const char* buf, size_t fsize, Qiniu_Io_PutExtra* extra);

// 根据文件名上传文件
Qiniu_Error Qiniu_Io_PutBuffer(
	Qiniu_Client* self, Qiniu_Io_PutRet* ret,
	const char* uptoken, const char* key, const char* buf, size_t fsize, Qiniu_Io_PutExtra* extra);
```

范围：客户端和服务端

## 断点续上传（resumable io）

```{c}
// “resumeable_io.h”

// upload

#ifndef QINIU_UNDEFINED_KEY
#define QINIU_UNDEFINED_KEY NULL
#endif

typedef struct _Qiniu_Rio_BlkputRet {
	const char* ctx;
	const char* checksum;
	Qiniu_Uint32 crc32;
	Qiniu_Uint32 offset;
	const char* host;
} Qiniu_Rio_BlkputRet;

typedef struct _Qiniu_Rio_PutExtra {
	const char* callbackParams;
	const char* bucket;
	const char* customMeta;
	const char* mimeType;
	int chunkSize;  // 可选。每次上传的Chunk大小
	int tryTimes;   // 可选。上传进度
	void* notifyRecvr;
	Qiniu_Rio_FnNotify notify;  // 进度提示。注意blk是并行传输的
	Qiniu_Rio_FnNotifyErr notifyErr;
	Qiniu_Rio_BlkputRet* progresses;  // 可选。上传进度
	size_t blockCnt;
	Qiniu_Rio_ThreadModel threadModel;
} Qiniu_Rio_PutExtra;

typedef Qiniu_Io_PutRet Qiniu_Rio_PutRet;

Qiniu_Error Qiniu_Rio_Put(
	Qiniu_Client* self, Qiniu_Rio_PutRet* ret,
	const char* uptoken, const char* key, Qiniu_ReaderAt f, Qiniu_Int64 fsize, Qiniu_Rio_PutExtra* extra);

Qiniu_Error Qiniu_Rio_PutFile(
	Qiniu_Client* self, Qiniu_Rio_PutRet* ret,
	const char* uptoken, const char* key, const char* localFile, Qiniu_Rio_PutExtra* extra);

int Qiniu_Rio_BlockCount(Qiniu_Int64 fsize);

// global settings

typedef struct _Qiniu_Rio_Settings {
	int taskQsize;     // 可选。任务队列大小。为 0 表示取 Workers * 4。 
	int workers;       // 并行的工作线程数目。
	int chunkSize;     // 默认的Chunk大小，不设定则为256k
	int tryTimes;      // 默认的尝试次数，不设定则为3
	Qiniu_Rio_ThreadModel threadModel;
} Qiniu_Rio_Settings;

void Qiniu_Rio_SetSettings(Qiniu_Rio_Settings* v);
```

范围：客户端和服务端

## 数据处理API（fop）

```{c}
// "fop.h"

// imageView
typedef struct _Qiniu_Fop_ImageView {
    int mode;
    int width;
    int height;
    int quality;
    const char* format;
} Qiniu_Fop_ImageView;

const char* Qiniu_Fop_ImageView_MakeRequest(Qiniu_Fop_ImageView* imageView, const char* url);

// imageInfo
typedef struct _Qiniu_Fop_ImageInfoRet {
    int width;
    int height;
    const char* format;
    const char* colorModel;
} Qiniu_Fop_ImageInfoRet;

Qiniu_Fop_ImageInfoRet* Qiniu_Fop_ImageInfo_Call(const char* url);

```

范围：客户端和服务端
