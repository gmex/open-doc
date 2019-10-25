# GMEX UCS Web API (v1)

## 说明

目前 GMEX (<https://www.gmex.io>) 对合作伙伴提供 Web API 用户资料操作开发接口， 供开发者进行第三方登录，调度资金等操作。


### GMEX官方的生产环境：

```txt
官方网址： https://www.gmex.io
用户授权服务： https://ucs-web.gmex.io/gaea/auth
用户出入金服务： https://ucs-web.gmex.io/gaea/trsf

```
流程说明图  

![Image text](https://github.com/gmex/open-doc/blob/master/desc_token.png)


### 接口请求格式说明：
请求方式为HTTP以POST方式请求，HTTP BODY为json格式，说明如下：
用户发送和接收到的所有消息统一采用JSON格式，发送请求的消息参数说明：


### 用户授权服务：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey|
|uid|用户在合作伙伴的账号体系中的uid|
|sign|请求消息的签名，签名方法如下：<br>   **md5($apiKey+$uid)**|


#### 请求回应数据示例：
```json
{
    "apiKeyId": "4SAAAB%67RhcZhZzD3JFZqRbABZA", 
    "uid": "666666", 
    "sign": "d6c42925bd10b0a1d98407280cba0728"
}

```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|token|发给用户的token，客户端可以用来出入金以及做币币和合约交易|
|reqUid|请求时提交的uid，方便数据对应|
|gaeaUId|用户在gaea账户体系中的UId，合作伙伴可以记录下来用来做后台对账|


#### 正确回应示例：
```json
{
    "code": 0, 
    "msg": "NO_ERROR", 
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==", 
    "reqUid": "666666",
    "gaeaUId": "1122333"
}
```

#### 错误回应示例：（例子：签名错误） 
```json
{
    "code": 7017, 
    "msg": "SIGN_ERROR"
}
```




### 用户出入金服务
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey|
|opType|**操作方式**：<br>**"dp"**:用户入金<br>**"wd"**:用户出金|
|token|用户在合作伙伴的账号体系中的uid|
|coin|用户在合作伙伴的账号体系中的uid|
|num|用户在合作伙伴的账号体系中的uid|
|sign|请求消息的签名，签名方法如下：<br>   **md5($apiKey+$token+$coin+$num)**|


#### 请求回应数据示例：
```json
{
    "apiKeyId" : "4SAAAB%67RhcZhZzD3JFZqRbABZA", 
    "opType" : "dp",
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==", 
    "coin" : "BTC",
    "num" : "0.00001",
    "sign": "d6c42925bd10b0a1d98407280cba0728"
}

```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|token|发给用户的token，客户端可以用来出入金以及做币币和合约交易|
|reqUid|请求时提交的uid，方便数据对应|
|gaeaUId|用户在gaea账户体系中的UId，合作伙伴可以记录下来用来做后台对账|


#### 回应示例：
```json
{
    "code": 0, 
    "msg": "NO_ERROR"
}
```




### 请求包的回应错误码说明
| ErrCode| ErrTxt | 描述 |
|:------:|:------|:------|
| 0          |  NORERROR                    | 没有错误 |
| 7016       |  JSON_UNMARSHAL_ERROR        | 请求json解码错误 |
| 7017       |  SIGN_ERROR                  | 签名错误 |
| 7018       |  SYSTEM_ERROR                | 服务器系统内部错误 |
| 7019       |  PROXY_API_KEY_ERROR         | 渠道号对应的apiKey不存在 |
| 7020       |  TOKEN_DATA_ERROR            | token的数据格式错误 |
| 7021       |  TOKEN_TIME_OUT              | token已超时，需要重新获取token |
| 7022       |  NOT_SUFFICIENT              | 资金余额不足（充值发生表示系统账户资金不足，出金发生表示用户账户资金不足） |

