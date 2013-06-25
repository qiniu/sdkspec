# SDK Implementation Specification


## 目录结构


```
docs/			- 文档所在目录
tests/  		- 单元测试目录，有的语言习惯叫 test 目录。这个建议遵循社区惯例。
README.md		- 项目说明文档
CHANGELOG.md	- 更新日志
.travis.yml		- Travis-CI 配置文件
```

## Travis-CI

不同语言的 .travis.yml 文件略有不同，但是他们都需要有：

```
before_script:
  - export QINIU_ACCESS_KEY="xxxxx"
  - export QINIU_SECRET_KEY="xxxxx"
```

然后在测试案例中通过 getenv("QINIU_ACCESS_KEY") 和 getenv("QINIU_SECRET_KEY") 获得 AccessKey/SecretKey。


## 开发流程

准备工作：

1. fork qiniu 中的某个 sdk（比如叫 foo-sdk）
2. git clone git@github.com:yourname/foo-sdk.git
3. git remote add qiniu git@github.com:qiniu/foo-sdk.git
4. git fetch qiniu

开发流程：

1. git checkout -b feature/yyyy qiniu/develop
2. 开发 (如果qiniu/develop分支别人有merge内容进来，可以随时执行 git pull 合并)
3. git commit -am "xxx"
4. git push origin feature/yyyy
5. 发起 pr

其他一些细节：

1. 如何同时开发多个feature。很简单：随时可以 git checkout feature/yyyy 到某个分支


## 更新日志

- 版本 Tag 创建的日期
- 引用 develop 到 master 的 pr
- 引用遵循的 sdkspec 版本
- 引用遵循的 apidoc 版本
- 说明本次修改的主要内容

样例：

```
## CHANGE LOG

### v5.0.0

2013-06-11 issue [#62](https://github.com/qiniu/api/pull/62)

- 遵循 [sdkspec v1.0.2](https://github.com/qiniu/sdkspec/tree/v1.0.2)
  - rs.GetPolicy 删除 Scope，也就是不再支持批量下载的授权。
  - rs.New, PutPolicy.Token, GetPolicy.MakeRequest 增加 mac *digest.Mac 参数。
- 遵循 [apidoc v3.0.0](https://github.com/qiniu/apidoc/tree/v3.0.0)
- 初步整理了 sdk 使用文档。
```


## 框架结构/名字空间

- conf：配置文件相关 (对于不支持名字空间的语言来说，建议用 `Qiniu_` 前缀做名字空间)
- auth.digest: 数字签名相关 (对于不支持名字空间的语言来说，建议用 `Qiniu_` 前缀做名字空间)
- rpc|http: 网络请求相关 (对于不支持名字空间的语言来说，建议用 `Qiniu_` 前缀做名字空间)
- rs: 上传下载凭证、文件管理相关 (对于不支持名字空间的语言来说，建议用 `Qiniu_RS_` 前缀做名字空间)
- rsf: 文件列表相关 (对于不支持名字空间的语言来说，建议用 `Qiniu_RS_` 前缀做名字空间)
- io: 普通上传相关 (对于不支持名字空间的语言来说，建议用 `Qiniu_RS_` 前缀做名字空间)
- resumable.io: 断点续上传相关 (对于不支持名字空间的语言来说，建议用 `Qiniu_RS_` 前缀做名字空间)
- fop: 数据处理相关 (对于不支持名字空间的语言来说，建议用 `Qiniu_FOP_` 前缀做名字空间)


## User-Agent 配置

- 建议 http 请求的 User-Agent 用 SDK名+版本。比如 "qiniu php-sdk v6.0.0"
