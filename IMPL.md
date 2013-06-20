# SDK Implementation Specification


## 目录结构


```
docs/			- 文档所在目录
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


## 框架结构

- conf：配置文件相关
- auth.digest: 数字签名相关
- rpc|http: 网络请求相关
- rs: 上传下载凭证、文件管理相关
- rsf: 文件列表相关
- io: 普通上传相关
- resumable.io: 断点续上传相关
- fop: 数据处理相关

