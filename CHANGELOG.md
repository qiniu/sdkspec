## CHANGE LOG

### v1.0.2

2013-06-10 issue [#21](https://github.com/qiniu/sdkspec/pull/21), [#22](https://github.com/qiniu/sdkspec/pull/22)

- rs.GetPolicy 删除 Scope，也就是不再支持批量下载的授权。
- rs.New, PutPolicy.Token, GetPolicy.MakeRequest 增加 mac *digest.Mac 参数，并默认为 nil。


### v1.0.1

2013-05-28

- io.GetUrl 改为 rs.MakeBaseUrl 和 rs.GetPolicy.MakeRequest
- rs.PutPolicy 增加 ReturnUrl, ReturnBody, CallbackBody；将 Customer 改为 EndUser；删除 CallbackBodyType（只支持 form 格式）

