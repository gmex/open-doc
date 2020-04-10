# GMEX CCS Web API (v1)

## 说明

目前 GMEX (<https://www.gmex.io>) 对合作伙伴提供 Web API 资金划转开发接口， 供开发者进行资金调度操作。


GMEX官方的生产环境：

```txt
官方网址： https://www.gmex.io
资金划转服务： https://ccs-web.gmex.io/gaea/transfer
```

## 资金划转API
请求方式为HTTP以POST方式请求，HTTP BODY为json格式，说明如下

请求回应数据示例
```json
{
    "channelId": "1", 
    "apiKeyId":"4SAAAB%67RhcZhZzD3JFZqRbABZA",
    "sign": "d6c42925bd10b0a1d98407280cba0728", 
    "batchReq": {
        "reqs": [
            {
                "seq": "20190429130352_1", 
                "coin": "GAEA", 
                "num": "100", 
                "acntFrom": {
                    "uid": "30000", 
                    "aType": 3
                }, 
                "acntTo": {
                    "uid": "1137338", 
                    "aType": 3
                }, 
                "info": "dp"
            }, 
            {
                "seq": "20190429130352_2", 
                "coin": "GAEA", 
                "num": "3111111111111111111111", 
                "acntFrom": {
                    "uid": "1137338", 
                    "aType": 3
                }, 
                "acntTo": {
                    "uid": "1137338", 
                    "aType": 5
                }, 
                "info": "transfer"
            }
        ]
    }
}

```
```json
{
    "channelId": "1", 
    "code": 0, 
    "msg": "NO_ERROR", 
    "resps": {
        "result": { }, 
        "resps": [
            {
                "result": { }, 
                "wallets": [
                    {
                        "wid": "1137338GAEA", 
                        "uid": "1137338", 
                        "coin": "GAEA", 
                        "exChannel": 1, 
                        "mainBal": "221333333334279635253.84911048000", 
                        "mainLock": "0.45000000000", 
                        "otcBal": "888888888888889352.79910000000", 
                        "otcLock": "81.60000000000", 
                        "financeBal": "138.50000000000"
                    }
                ]
            }, 
            {
                "result": {
                    "code": 8002, 
                    "msg": "NOT_SUFFICIENT"
                }
            }
        ]
    }
}
```

用户发送和接收到的所有消息统一采用JSON格式，发送请求的消息参数说明：


|请求包体参数| 描述|
| :---  | :---|
|channelId|合作伙伴的渠道id，接入之前，请先联系我方运营索取资金的渠道Id|
|apiKeyId|合作伙伴的apiKeyId，接入之前，请先联系我方运营索取用于签名的apiKeyId和apiKey|
|sign|请求消息的签名，签名方法如下：<br>  md5(apiKey + [req.AcntFrom.Uid + req.AcntTo.Uid + req.Num + req.Coin])<br>方括号本身不用加入签名，方括号里面里为batchReq里面的所有请求循环累加|
|batchReq|存放的请求批处理列表|


转账请求的结构定义如下：
```golang
// rpc请求： 处理用户之间转账相关业务
type TransferReq struct {
	Seq         string      // 【必填字段】见补充说明： 【CCS订单号补充规则】
	AcntFrom    *CcsAccount // 【必填字段】转出方
	AcntTo      *CcsAccount // 【必填字段】收款方
	Coin        string      // 【必填字段】要划转的币种
	Num         string      // 【必填字段】要划转的数量
	Info        string      // 【必填字段】操作信息，必须是英文，长度不超过20字符

	PeerSeq     string      // 【选填】可以带一个对应自己系统的订单号以供查询
	Fee         string      // 【选填】费用1（从转出者账上扣除）
	Fee2        string      // 【选填】费用2（从收款者账上扣除）
	FeeUId      string      // 【选填】收取费用的账号UId（收取的费用将会保存到该账户上）
	CcsPasswd   string      // 【选填】如果是用户间转账需带密码（代理服务器可以自动添加）
	RecordType  int32       // 【选填】保存资金记录的栏目（默认值0不记录，1-充值历史，2-提现历史，3-手续费返还，4-资金划转，5-其他）
}

```

|回应包体参数| 描述|
| :-----   | :-----   |
|channelId|合作伙伴的渠道id，回传给请求方|
|code|请求错误码，见下方【请求包的回应错误码说明】|
|msg|错误码对应的字符串说明|
|resps|对应批处理的每条请求的回应<br>如果该笔转账成功，将会返回受影响的钱包信息，见示例第一条转账回应<br>如果该笔转账失败，将会返回错误码和错误信息，见示例第二条转账回应|

### 订单号规则说明
|CCS订单号规则| 描述|
| :-----   | :-----   |
| 目的|订单号必须以固定格式的日期开头，资金中心可以丢弃过时的订单，不必保存所有订单号来防止重放攻击，释放服务器的内存占用|
|推荐格式|"20190320053833057dm9_TOUTL"，即【年月日时分秒微秒自增号_分类信息】|
|超时错误|必须以YYYYMMDD日期格式开头，如2019-03-20的订单必须以20190320开头，日期超过3天的订单会返回订单号超时错误【8016】|
|格式错误|"TOUTL_20190320053833057"，不符合格式的订单会返回订单格式错误【8015】|


### 转账的账户类型说明（aType字段）
| aType| 描述 |
|:------|:------|
| 3       | 钱包流通账户 |
| 4       | 锁定币（矿池） |
| 5       | OTC账户 | 
| 6       | OTC锁定 |
| 7       | 理财账户 |
| 8       | OTC账户 |
| 15      | 充值锁定 |

### 请求包的回应错误码说明
| ErrCode| ErrTxt | 描述 |
|:------:|:------|:------|
| 0          |  NORERROR                    | 没有错误 |
| 8017       |  PROXY_JSON_UNMARSHAL_ERROR  | 请求json解码错误 |
| 8018       |  PROXY_SIGN_ERROR            | 签名错误 |
| 8019       |  PROXY_SYSTEM_ERROR          | 系统内部错误 |
| 8020       |  PROXY_TOO_MANY_REQS         | 请求批次数量过多，同一批次不可超过100条 |
| 8021       |  PROXY_API_KEY_ERROR         | 渠道号对应的apiKey不存在 |


### 批处理内部的请求回应错误码说明
| ErrCode| ErrTxt | 描述 |
|:------:|:------|:------|
| 0          |  NORERROR                | 没有错误 |
| 8002       |  NOT_SUFFICIENT          | 资金余额不足 |
| 8003       |  SEQ_EXISTS              | 订单号已存在（重复请求） |
| 8007       |  UNSUPPORTED_OP          | 不支持的操作 |
| 8008       |  SYSTEM_ERROR            | 系统出错 |
| 8009       |  INPUT_ERROR             | 输入错误 |
| 8013       |  PASSWD_INVALID          | 资金密码错误 |
| 8015       |  ORDER_FORMAT_ERROR      | 订单格式错误 |
| 8016       |  ORDER_FORMAT_TIME_OUT   | 订单号超时错误 |

