# GMEX UCS Web API (v1)

## 说明

目前 GDO (<https://www.gmex.me>) 对合作伙伴提供 Web API 用户资料操作开发接口， 供开发者进行第三方登录，调度资金等操作。


### GMEX官方的生产环境：

```txt
官方网址： https://www.gmex.me

正式环境接口地址（国内访问）： https://ucs-web.gmex.me/
正式环境接口地址（国外访问）： https://ucs-web.gmex.io/
测试环境接口地址： http://eeeecloud.com:8086/

必需接口：
1.用户授权服务： https://ucs-web.gmex.io/gaea/auth
2.用户出入金服务： https://ucs-web.gmex.io/gaea/trsf

可选接口：
3.用户登出服务： https://ucs-web.gmex.io/gaea/lgot
4.token验证服务： https://ucs-web.gmex.io/gaea/chktkn
5.用户切换语言服务： https://ucs-web.gmex.io/gaea/lang
6.用户资产查询服务： https://ucs-web.gmex.io/gaea/qast
7.修改用户数据服务： https://ucs-web.gmex.io/gaea/mdfus
8.出入金订单结果查询： https://ucs-web.gmex.io/gaea/trsfqry


```

流程说明图  

![Image text](https://github.com/gmex/open-doc/blob/master/desc_token.png?raw=true)


### 接口请求格式说明：
请求方式为HTTP以POST方式请求，HTTP BODY为json格式，说明如下：
用户发送和接收到的所有消息统一采用JSON格式，发送请求的消息参数说明：


### 1.用户授权服务接口：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey。测试代码用的apiKey见文档末表格|
|uid|用户在合作伙伴的账号体系中的uid|
|timeOut|**选填参数**，token有效时间（秒），不传默认15天有效期|
|email|【**选填参数，仅首次关联账号时有效**】：用户的邮件地址，可以用来发送爆仓或风险预警通知|
|phone|【**选填参数，仅首次关联账号时有效**】：用户的电话号码，可以用来发送爆仓或风险预警通知，格式示例：0086-13812345678|
|upd|【**废弃参数，请使用修改用户资料接口来处理**】：用来指定是否修改用户数据（首次关联时无需此参数），0-默认值，不更新，1-更新用户邮箱，2-更新手机号
|ts|unix时间戳（秒），请尽量保证时间准确，误差过大（比如超过10分钟），请求有可能会被服务器丢弃|
|sign|请求消息的签名，签名方法如下：<br>  **md5($apiKey+$uid+$pUid+$email+$phone+$ts)**|


#### 请求回应数据示例：
```json
{
    "apiKeyId": "uTQAAg0$unGYzC9qYsznB3bCBBaE", 
    "uid": "666666",
    "timeOut": 360000, 
    "email": "xxx@gmail.com", 
    "phone": "0086-13812345678", 
    "ts": 1586502220,
    "sign": "32e786c6d993e011140b102bf8684253"
}

```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|token|发给用户的token，客户端可以用来出入金以及做币币和合约交易|
|reqUid|请求时提交的uid，方便数据对应|
|gaeaUId|用户在GDO账户体系中的UId，合作伙伴可以记录下来用来做后台对账|


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

### 2.用户出入金服务
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey，测试代码用的apiKey见文档末表格|
|opType|**操作方式**： <br> **"dp01"**:合约入金  <br>**"dp02"**:币币入金<br>  **"wd01"**:合约出金<br>  **"wd02"**:币币出金|
|token|需要处理出入金的用户，通过授权接口获得的token|
|gaeaUid|需要处理出入金的用户，在GDO系统内的Uid，优先取token参数，如果token参数不为空的话，此字段不生效|
|reqOrderId|请求操作的订单号，不可重复，可用来防止重复操作以及对账，就算操作失败也不能重复使用，如果传空的值，则不会进行订单号相关的检测 <br> **"推荐格式**: "201903200538330572359_ABCDE"，即【年月日时分秒微秒自增号_分类信息】|
|coin|出入金的币种（全部大写）|
|num|出入金的数量（精度说明：整数位无限制，小数位最多8位，小数超过8位会报错：INPUT_PARA_ERROR）|
|sign|请求消息的签名，签名方法如下：<br>   **md5($apiKey+$opType+$token+$gaeaUid+$reqOrderId+$coin+$num)**|


#### 请求数据示例：
```json
{
    "apiKeyId" : "uTQAAg0$unGYzC9qYsznB3bCBBaE", 
    "opType" : "dp01",
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==", 
    "gaeaUid": "112233", 
    "reqOrderId" : "201903200538330572359_ABCDE",
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
    "reqOrderId" : "201903200538330572359_ABCDE",
    "orderId": "20191031165959003xzhLGIFT"
}
```

<span style="color: crimson">
#### 测试注意事项：
**注意出入金接口一定要进行并发测试!**<br>
1.假设用户合约账上有100USDT，同时并发发起几十笔出金交易，看下处理流程和最后的资金情况是否正常。<br>
2.假设用户在合作方账上有100USDT，同时并发发起几十笔入金交易，看下处理流程和最后的资金情况是否正常。<br>
</span>

### 3.用户登出服务接口：
#### 当用户在合作方账户体系中退出时，可以回调此接口，我方服务器将会断开用户交易的web socket连接，保证用户确实退出。
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey，测试代码用的apiKey见文档末表格|
|token|用户通过授权接口获得的token|
|ts|unix时间戳（秒），请尽量保证时间准确，误差过大（比如超过10分钟），请求有可能会被服务器丢弃|
|sign|请求消息的签名，签名方法如下：<br>   **md5($apiKey+$token+$ts)**|


#### 请求数据示例：
```json
{
    "apiKeyId":"uTQAAg0$unGYzC9qYsznB3bCBBaE",
    "token":"GQAADdmxxo1sbNg7MZhizIGGjcmzM7nOcJ6F5XaaO8QHTA==",
    "ts":1586503865,
    "sign":"d4cac81384e183d3878391ef2384f33a"
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


### 4.验证token服务接口：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey，测试代码用的apiKey见文档末表格|
|token|用户通过授权接口获得的token|

#### 请求数据示例：
```json
{
    "apiKeyId": "uTQAAg0$unGYzC9qYsznB3bCBBaE", 
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==" 
}
```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|token|如果token是有效的话，回发请求的token|
|gaeaUId|用户在GDO账户体系中的UId|
|leftSeconds|如果token是有效的话，token剩余的有效时间，单位秒|

#### token有效，回应数据示例：
```json
{
    "code": 0, 
    "msg": "NO_ERROR", 
    "token": "PgAAmDupoZkMkYljZjIgxbhcjXtrO4mNpjK5xphnaGQ7HHqxzXYTs8gKRg==", 
    "GaeaUId": "1122333",
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

### 5.用户切换语言服务接口：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey，测试代码用的apiKey见文档末表格|
|token|用户通过授权接口获得的token|
|lang|用户切换到的目标语言的字母缩写，<br>已定义的语言缩写：中文-zh， 台湾-tw， 日语-jp， 英语-en， 韩语-kr， 法语-fr， 俄语-ru， 德语-de， 越南语-vn， 西班牙语-es， 葡萄牙语-pt， 阿拉伯语-ar， 印度语-in， 土耳其语-tr， 泰语-th


#### 请求数据示例：
```json
{
    "apiKeyId": "uTQAAg0$unGYzC9qYsznB3bCBBaE", 
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

### 6.用户查询资产：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey，测试代码用的apiKey见文档末表格|
|token|用户通过授权接口获得的token|
|assetType|用户查询资产种类： 合约资产-"01"，币币资产-"02"， 查询系统账户钱包余额-"99"（此时不用传token）
|ts|unix时间戳（秒），请尽量保证时间准确，误差过大（比如超过10分钟），请求有可能会被服务器丢弃|
|sign|请求消息的签名，签名方法如下：<br>   **md5($apiKey+$token+$assetType+$ts)**|


#### 请求数据示例：
```json
{
    "apiKeyId":"uTQAAg0$unGYzC9qYsznB3bCBBaE",
    "token":"OwAAaJ0bNhsbbBt2YpiJmQM0G5M7jM12HJ6YYDfszsQHXg==",
    "assetType":"01",
    "ts":1586506720,
    "sign":"247963494754ec1ca3857650a7639853"
}
```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|token|回传请求的token|
|wlts|钱包数据（具体含义以及处理见前端标准版代码）|
|poss|持仓数据（具体含义以及处理见前端标准版代码）|
|mainBalance|系统母账户当前余额|

#### 操作有效，回应数据示例：
```json
{
    "code": 0,
    "msg": "NO_ERROR",
    "token": "",
    "wlts": [
        {
            "UId": "1137099",
            "AId": "113709901",
            "Coin": "USDT",
            "WId": "113709901USDT",
            "Depo": 0,
            "WDrw": 0,
            "PNL": 0,
            "Frz": 0,
            "Spot": 0,
            "Gift": 0,
            "PNLG": 0,
            "Status": 2,
            "Flg": 6144,
            "DF": 289
        }
    ],
    "poss": [
        {
            "UId": "1024907",
            "PId": "01E38V9EDFWJE2MPGCMA11ZM4Z",
            "AId": "102490701",
            "Sym": "BCH.USDT@21",
            "WId": "102490701USDT",
            "Sz": 2400.00,
            "PrzIni": 136.53,
            "RPNL": 15089.074606456912,
            "MgnISO": 0,
            "PNLISO": 0,
            "LeverMax": 100,
            "MMR": 0.005,
            "MIR": 0.01,
            "Val": 16383.6,
            "MMnF": 81.918,
            "UPNL": 10230.939530523603,
            "PrzLiq": 0.01,
            "PrzBr": 0.01,
            "FeeEst": 13.3072697652618,
            "MIRMy": 0.01,
            "ROE": 0.6244622384899291,
            "ADLIdx": 0.6244903955985741,
            "ADLLight": 4,
            "DF": 288
        },
        {
            "UId": "1024907",
            "PId": "01E38TSD43YZA4A49T4GHRM26R",
            "AId": "102490701",
            "Sym": "ETH.USDT@21",
            "WId": "102490701USDT",
            "Sz": 4970.00,
            "PrzIni": 88.82,
            "RPNL": 0.16063636006226167,
            "MgnISO": 0,
            "PNLISO": 0,
            "LeverMax": 100,
            "MMR": 0.005,
            "MIR": 0.01,
            "Val": 44143.5399999,
            "MMnF": 220.71769988,
            "UPNL": 24176.229983015328,
            "PrzLiq": 0.01,
            "PrzBr": 0.01,
            "FeeEst": 34.15988499150767,
            "MIRMy": 0.01,
            "ROE": 0.5476731132803425,
            "ADLIdx": 0.5477129572874402,
            "ADLLight": 4,
            "DF": 288
        }
    ],
	"mainBalance": ["BTC:19995878.99913000000", "USDT:198019999978891.00000000000", "ETH:200000000000000.00000000000"]
}

```



### 7.修改用户数据服务接口：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey。测试代码用的apiKey见文档末表格|
|opType|**操作方式**： <br> **1**:设置上级关系  <br>**2**:修改用户手机号<br>  **3**:修改用户邮箱|
|gaeaUid|用户在GDO的账号体系中的uid|
|pGaeaUid|【**选填参数**】，上级用户的gaeaUid，只能在没有关系时设置，如果传必须是有效uid，否则报错|
|email|【**选填参数**】：用户的邮件地址，可以用来发送爆仓或风险预警通知|
|phone|【**选填参数**】：用户的电话号码，可以用来发送爆仓或风险预警通知，格式示例：0086-13812345678|
|ts|unix时间戳（秒），请尽量保证时间准确，误差过大（比如超过10分钟），请求有可能会被服务器丢弃|
|sign|请求消息的签名，签名方法如下：<br>  **md5($apiKey+$opType+$gaeaUid+$pGaeaUid+$email+$phone+$ts)**|


#### 请求回应数据示例：
```json

{
  "apiKeyId":"uTQAAg0$unGYzC9qYsznB3bCBBaE",
  "opType":1,
  "gaeaUid":"666666",
  "pGaeaUid":"111",
  "email":"xxx@gmail.com",
  "phone":"0086-13812345678",
  "ts":1589288384,
  "sign":"a9a63d4ba12e038a77e252beafba65d1"
}

```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|reqUid|请求时提交的gaeaUid，方便数据对应|



#### 正确回应示例：
```json
{
    "code": 0, 
    "msg": "NO_ERROR", 
    "reqUid": "666666"
}
```

#### 错误回应示例：（例子：用户不存在） 
```json
{
  "code":7027,
  "msg":"用户不存在",
  "reqUid":"666666"
}
```


### 8.出入金订单结果查询：
|请求包体参数| 描述|
| :---  | :---|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey。测试代码用的apiKey见文档末表格|
|reqOrderId|要查询的订单号，即出入金服务请求的订单号|
|sign|请求消息的签名，签名方法如下：<br>  **md5($apiKey+$reqOrderId)**|

#### 重要说明：
如果订单号重复，不会保存重复的订单号处理状态，防止正确处理的订单号状态被覆盖。<br>
如果有重复订单号的嫌疑查询时，请注意返回结果中的ts（订单被处理的时间戳）可以协助判断。<br>
为了防止正确的订单号状态被覆盖，在出入金操作时，订单号防止重复流程判断之前出错后，不会保存订单被处理的状态，此类订单号查询结果应该是订单号不存在：QUERY_ORDER_ID_NOT_EXISTS。<br>
不会被保存的状态，包括但不限于以下错误状态：JSON_UNMARSHAL_ERROR， API_KEY_ERROR， SIGN_ERROR， INPUT_PARA_ERROR， ORDER_ID_EXISTS<br>
**注意**：订单号是否被正确处理的判断依据是：查询结果的code为0，并且返回结果里respResult字段里面的code为0

#### 请求回应数据示例：
```json
{
	"apiKeyId": "uTQAAg0$unGYzC9qYsznB3bCBBaE",
	"reqOrderId": "201903200538330572359_ABCDE",
	"sign": "54d1f0a196b8d297b2add8e4e500c078"
}

```

|回应包体参数| 描述|
| :-----   | :-----   |
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|respResult|在出入金请求操作时，返回的结果，以字符串形式保存|
|peerOrders|在出入金操作时，我方关联的订单号，可以用来在后台查询相关数据|
|ts|出入金操作被处理时的时间戳，**注意不是查询的时间戳**|


#### 正确回应示例：
```json
{
	"code": 0,
	"msg": "NO_ERROR",
	"respResult": "{\"code\":0,\"msg\":\"NO_ERROR\",\"token\":\"PAAAQ9ljGxs22GzOMZgxZhiGNjk7M53iyCPL6gbZYgbL\",\"reqOrderId\":\"201903200538330572359_ABCDE\",\"orderId\":\"20200603153924007mquAPIDP\"}",
	"peerOrders": ["20200603153924007mquAPIDP", "20200603153924007mqvTOUT/20200603153924007mqwTIN"],
	"ts": 1591169964
}
```

#### 错误回应示例：（例子：订单不存在） 
```json
{
	"code": 7033,
	"msg": "QUERY_ORDER_ID_NOT_EXISTS",
	"respResult": "",
	"peerOrders": null,
	"ts": 0
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
| 7027       |  USER_NOT_EXISTS             | 出入金或设置用户资料操作，用户不存在 |
| 7028       |  PARENT_NOT_EXISTS           | 设置上级关系时，上级用户不存在 |
| 7029       |  PARENT_UID_EXISTS           | 设置上级关系时，关系已经存在 |
| 7030       |  PARENT_RELATION_ERROR       | 设置上级关系时，关系错误（如上级跟下级不是一个渠道的用户） |
| 7031       |  PARENT_FORBIDDEN            | 设置上级关系时，上级用户被禁止 |
| 7032       |  PERMISSION_ERROR            | 没有权限，比如操作其他渠道用户 |
| 7033       |  QUERY_ORDER_ID_NOT_EXISTS   | 出入金结果查询操作，订单号不存在 |


### 测试用apiKey


| 字段 | 内容 | 
|:------:|:------|
|测试APIKeyId| uTQAAg0$unGYzC9qYsznB3bCBBaE|
|测试APIKey| 6UAAAHHruk2bMLLZMZ8nBu1gGPvi6uenNwTRYLyzksmdX0OSG3Wa44g5V | 
|测试渠道号| 3 |
| 说明 | 传参数用APIKeyId，签名用APIKey |  
