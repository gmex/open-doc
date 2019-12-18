# GMEX UCS Web API (v1)

## 说明

目前 GMEX (<https://www.gmex.io>) 对合作伙伴提供 Web API 用户资料操作开发接口， 供开发者进行第三方登录，调度资金等操作。


### GMEX官方的生产环境：

```txt
官方网址： https://www.gmex.io
用户授权服务： https://ucs-web.gmex.io/gaea/auth
token验证服务： https://ucs-web.gmex.io/gaea/chktkn
用户切换语言服务： https://ucs-web.gmex.io/gaea/lang
用户出入金服务： https://ucs-web.gmex.io/gaea/trsf

```
流程说明图  

![Image text](https://github.com/gmex/open-doc/blob/master/desc_token.png?raw=true)


### 接口请求格式说明：
请求方式为HTTP以POST方式请求，HTTP BODY为json格式，说明如下：
用户发送和接收到的所有消息统一采用JSON格式，发送请求的消息参数说明：


### 用户授权服务接口：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey|
|uid|用户在合作伙伴的账号体系中的uid|
|email|【**选填参数，可在首次关联账号时传**】：用户的邮件地址，可以用来发送爆仓或风险预警通知|
|phone|【**选填参数，可在首次关联账号时传**】：用户的电话号码，可以用来发送爆仓或风险预警通知，格式示例：0086-13812345678|
|upd|【**选填参数，可在更改用户信息时传**】：用来指定是否修改用户数据（首次关联时无需此参数），0-默认值，不更新，1-更新用户邮箱，2-更新手机号
|sign|请求消息的签名，签名方法如下：<br>   **md5($apiKey+$uid)**|


#### 请求回应数据示例：
```json
{
    "apiKeyId": "4SAAAB%67RhcZhZzD3JFZqRbABZA", 
    "uid": "666666", 
    "email": "xxx@gmail.com", 
    "phone": "0086-13812345678", 
    "upd": 0, 
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


### 验证token服务接口：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey|
|token|用户通过授权接口获得的token|

#### 请求数据示例：
```json
{
    "apiKeyId": "4SAAAB%67RhcZhZzD3JFZqRbABZA", 
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==" 
}
```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|token|如果token是有效的话，回发请求的token|
|leftSeconds|如果token是有效的话，token剩余的有效时间，单位秒|

#### token有效，回应数据示例：
```json
{
    "code": 0, 
    "msg": "NO_ERROR", 
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==", 
    "leftSeconds": 1234
}
```
#### token无效，回应数据示例：
```json
{
    "code": 7020, 
    "msg": "TOKEN_DATA_ERROR"
}
```

### 用户切换语言服务接口：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey|
|token|用户通过授权接口获得的token|
|lang|用户切换到的目标语言的字母缩写，<br>已定义的语言缩写：中文-zh， 台湾-tw， 日语-jp， 英语-en， 韩语-kr， 法语-fr， 俄语-ru， 德语-de， 越南语-vn， 西班牙语-es， 葡萄牙语-pt， 阿拉伯语-ar， 印度语-in， 土耳其语-tr， 泰语-th
|

#### 请求数据示例：
```json
{
    "apiKeyId": "4SAAAB%67RhcZhZzD3JFZqRbABZA", 
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==",
    "lang": "zh"  
}
```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|

#### 操作有效，回应数据示例：
```json
{
    "code": 0, 
    "msg": "NO_ERROR" 
}
```


### 用户出入金服务
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey|
|opType|**操作方式**： <br> **"dp01"**:合约入金  <br>**"dp02"**:币币入金<br>  **"wd01"**:合约出金<br>  **"wd02"**:币币出金|
|token|需要处理出入金的用户，通过授权接口获得的token|
|gaeaUid|需要处理出入金的用户，在gaea系统内的Uid，优先取token参数，如果token参数不为空的话，此字段不生效|
|reqOrderId|请求操作的订单号，不可重复，可用来防止重复操作以及对账，就算操作失败也不能重复使用，如果传空的值，则不会进行订单号相关的检测|
|coin|出入金的币种（全部大写）|
|num|出入金的数量（精度说明：整数位无限制，小数位最多8位）|
|sign|请求消息的签名，签名方法如下：<br>   **md5($apiKey+$opType+$token+$gaeaUid+$reqOrderId+$coin+$num)**|


#### 请求数据示例：
```json
{
    "apiKeyId" : "4SAAAB%67RhcZhZzD3JFZqRbABZA", 
    "opType" : "dp01",
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==", 
    "gaeaUid": "112233", 
    "reqOrderId" : "20191031165959003xzhLxd2312232",
    "coin" : "BTC",
    "num" : "1.00002",
    "sign": "d6c42925bd10b0a1d98407280cba0728"
}

```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|token|请求转账用户的token，方便数据对应|
|orderId|请求转账成功后的账单流水号，方便对账用|


#### 回应示例：
```json
{
    "code": 0, 
    "msg": "NO_ERROR",
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==",
    "orderId": "20191031165959003xzhLGIFT"
}
```




### 请求包的回应错误码说明
| ErrCode| ErrTxt | 描述 |
|:------:|:------|:------|
| 0          |  NO_ERROR                    | 没有错误 |
| 7016       |  JSON_UNMARSHAL_ERROR        | 请求json解码错误 |
| 7017       |  SIGN_ERROR                  | 签名错误 |
| 7018       |  SYSTEM_ERROR                | 服务器系统内部错误 |
| 7019       |  API_KEY_ERROR               | 渠道号对应的apiKey不存在 |
| 7020       |  TOKEN_DATA_ERROR            | token的数据格式错误 |
| 7021       |  TOKEN_TIME_OUT              | token已超时，需要重新获取token |
| 7022       |  NOT_SUFFICIENT              | 资金余额不足（充值发生表示系统账户资金不足，出金发生表示用户账户资金不足） |
| 7023       |  EMAIL_FORMAT_ERROR          | 邮件格式错误 |
| 7024       |  PHONE_FORMAT_ERROR          | 手机号格式错误，示例：0086-13812345678 |
| 7025       |  INPUT_PARA_ERROR            | 参数输入错误(比如数字输入了字符串，必填参数为空等) |
| 7026       |  ORDER_ID_EXISTS             | 出入金操作，订单号重复 |
| 7027       |  USER_NOT_EXISTS             | 出入金操作，用户不存在 |

