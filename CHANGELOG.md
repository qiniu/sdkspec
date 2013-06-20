## CHANGE LOG

### v6.0.0

2013-06-21 issue [#27](https://github.com/qiniu/sdkspec/pull/27), [#28](https://github.com/qiniu/sdkspec/pull/28)

- 增加 [SDK 实现规范](https://github.com/qiniu/sdkspec/blob/develop/IMPL.md)
- 增加 conf.USER_AGENT 变量


### v1.0.3

2013-06-20 issue [#25](https://github.com/qiniu/sdkspec/pull/25)

- io.PutExtra
  - Crc32        uint32
  - CheckCrc     uint32
    // CheckCrc == 0: 表示不进行 crc32 校验
    // CheckCrc == 1: 对于 Put 等同于 CheckCrc = 2；对于 PutFile 会自动计算 crc32 值
    // CheckCrc == 2: 表示进行 crc32 校验，且 crc32 值就是上面的 Crc32 变量


### v1.0.2

2013-06-10 issue [#21](https://github.com/qiniu/sdkspec/pull/21), [#22](https://github.com/qiniu/sdkspec/pull/22)

- rs.GetPolicy 删除 Scope，也就是不再支持批量下载的授权。
- rs.New, PutPolicy.Token, GetPolicy.MakeRequest 增加 mac *digest.Mac 参数，并默认为 nil。


### v1.0.1

2013-05-28

- io.GetUrl 改为 rs.MakeBaseUrl 和 rs.GetPolicy.MakeRequest
- rs.PutPolicy 增加 ReturnUrl, ReturnBody, CallbackBody；将 Customer 改为 EndUser；删除 CallbackBodyType（只支持 form 格式）

