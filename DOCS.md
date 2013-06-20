# docs.qiniu.com Specification

## 网站结构

- guide: QuickStart向导文档
- api: 协议文档
- xxx-sdk: SDK帮助文档（包含Tech文档，未来可能包含Reference文档）
- tools: 工具帮助文档（包含 qrsync, qrsctl，etc）

结构如下：

```
/guide
	/v1/xxx
	/v2/xxx
	/v3/xxx

/api
	/v1/xxx
	/v2/xxx
	/v3/xxx

/php-sdk
	/v1/xxx
	/v2/xxx
	/v3/xxx

/python-sdk
	/v1/xxx
	/v2/xxx
	/v3/xxx

/tools
	/v1/xxx
	/v2/xxx
	/v3/xxx
```


## 网站维护

- api 文档，在 https://github.com/qiniu/apidoc 中维护。
- xxx-sdk 文档，在各个 SDK 本身的 repo 中维护，一般在 repo 下的 docs 目录。
- 其他，在 https://github.com/qiniu/docs.qiniu.com 中维护。


## 版本管理

根据 master 分支下的 CHANGELOG 确定当前版本。

