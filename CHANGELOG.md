## CHANGE LOG

### v1.0.1

2013-05-28 issue [#56](https://github.com/qiniu/api/pull/56)

- io.GetUrl 改为 rs.MakeBaseUrl 和 rs.GetPolicy.MakeRequest
- rs.PutPolicy 增加 ReturnUrl, ReturnBody, CallbackBody；将 Customer 改为 EndUser；删除 CallbackBodyType（只支持 form 格式）

