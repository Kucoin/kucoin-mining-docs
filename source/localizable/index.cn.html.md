# 快速开始
## 简介
欢迎使用KuCoin矿池(KuCoin Pool)开发者文档。 此文档概述了用户获取矿池数据信息相关的开发接口。

# REST API
## API服务器地址
基本URL: https://api.kucoin.com/mining-pool
<aside class="notice">为了遵守当地法律要求，使用中国IP的用户不允许访问以上URL。</aside>

请求URL由基本URL和接口端点组成。

## 接口端点(Endpoint)
每个接口都提供了对应的端点（Endpoint），可在HTTP请求模块下获取。

对于GET 请求, 端点需要要包含请求参数。

例如，对于"收益列表"接口，其默认端点为 /v1/external/earn/query。 如果您的请求参数algo=SHA256d，则该端点将变为 /v1/external/earn/query?algo=SHA256d。因此，您最终请求的URL应为：https://api.kucoin.com/mining-pool/v1/external/earn/query?algo=SHA256d。

## 请求
- 所有的POST请求都是 application/json.
- 所有的请求返回内容类型都是 application/json.

除非另行说明，所有的时间戳参数均以Unix时间戳毫秒计算。如：1544657947759

### 请求参数
对于GET请求, 需将参数拼接在请求URL中。(例如，/v1/external/earn/query?algo=SHA256d)

对于 POST和PUT 请求, 需将参数以JSON格式拼接在请求主体中(例如，{"algo":"SHA256d"})。 注意：不要在JSON字符串中添加空格。

### 错误返回
系统会返回HTTP错误代码或系统错误代码。您可根据返回的参数消息排查错误原因。

### HTTP错误码
代码 |	意义
---|---
400 | Bad Request -- 无效的请求格式
401 | Unauthorized -- 无效的API-KEY
403 | Forbidden 或 Too Many Requests -- 请求被禁止 或 超过请求频率限制
404 | Not Found -- 找不到指定资源
405 | Method Not Allowed -- 您请求资源的方法不正确
415 | Content-Type: application/json -- 请求类型必须为application/json类型
500 | Internal Server Error -- 服务器出错，请稍后再试
503 | Service Unavailable -- 服务器维护中，请稍后再试

### 系统错误码
代码 | 意义
---|---
200002 | Please try again later -- 频繁请求，接口进行了限流
400001 | Any of key, timestamp, passphrase, version is missing in your request parameter -- 请求参数中缺少验签参数
400002 | Invalid timestamp -- 请求时间与服务器时差超过5秒
400003 | Key not exists -- Api-key 不存在
400004 | Invalid passphrase -- passphrase 不正确
400100 | Parameter Error -- 请求参数不合法
500000 | Internal Server Error -- 服务器出错，请稍后再试

如果系统返回HTTP状态码为200，但业务失败，系统会报错。您可根据返回的参数消息排查错误。

### 接口返回
api接口返回统一json格式。

当系统返回HTTP状态码200和系统代码200000时，表示响应成功，返回结果如下：

- 普通请求返回

```json
{
  "success": true,
  "code": "200", //状态码
  "msg": "success", //错误信息
  "retry": false,
  "data": "" //返回数据。任意类型，代表返回的数据
}
```
- 分页请求返回

```json
{
  "code": "200", //状态码
  "data": {      //返回数据。任意类型，代表返回的数据
    "currentPage": 1, //当前页
    "items": [], //数组数据
    "pageSize": 10, //每页数量
    "totalNum": 100, //总条数
    "totalPage": 10 //总页数
  },
  "msg": "success", //错误信息
  "retry": false,
  "success": true
}
```


### 分页
Pagination允许使用当前页数获取结果，非常适用于获取实时数据。如 /v1/external/worker/query、/v1/external/earn/query端点均默认返回第一页结果，共50条数据。如需获取更多数据，请根据当前返回的数据指定其他分页，然后再进行请求。

#### 请求参数
参数名称 | 默认值 | 含义
---|---|---
currentPage | 1 | 当前页码
pageSize | 50 | 每页记录数，最小值10，最大值500

示例
GET /v1/external/worker/query?currentPage=1&pageSize=50



# 接口认证
## 创建API-KEY
通过接口进行请求前，您需在官网矿池->设置->API管理创建API-KEY。创建成功后，您需妥善保管好以下三条信息：

- Api-Key
- Api-Secret
- Passphrase

Key和Secret由KuCoin随机生成并提供，Passphrase是您在创建API时使用的密码。以上信息若遗失将无法恢复，需要重新申请API KEY。

## 创建请求
Rest请求地址参数中必须包含以下内容:

- key: Api-Key以字符串传递
- timestamp: 请求的时间戳
- passphrase: 创建API时填的API-Key的密码
- version: Api-Key版本号, 当前版本号约定=2


请求参数中的**passphrase:**

1. 将passphrase使用Api-Secret进行**HMAC-sha256**加密，再将加密内容通**过base64**编码后传递


```
    #Example for get algo list in python

    api_key = "api_key"
    api_secret = "api_secret"
    api_passphrase = "api_passphrase"
    url = 'https://api.kucoin.com/mining-pool/v1/external/algo/query'
    now = int(time.time() * 1000)

    passphrase = base64.b64encode(hmac.new(api_secret.encode('utf-8'), api_passphrase.encode('utf-8'), hashlib.sha256).digest())
    
    sign_parameters = 'timestamp=' + str(now) + '&key=' + api_key + '&passphrase=' + passphrase + '&version=' + 2

    url = url + '?' + sign_parameters
    response = requests.request('get', url)
    print(response.status_code)
    print(response.json())

```


# 算法模块
## 算法列表
获取kuCoin矿池下支持挖矿的算法。

**请求频率：** api-key级别每秒2次


#### HTTP请求
GET /v1/external/algo/query

#### 请求示例
GET /v1/external/algo/query

#### 返回值
参数 | 类型 | 含义
---|---|---
data | list | 算法名称

### 返回示例

```json
{
  "success": true,
  "code": "200",
  "msg": "success",
  "retry": false,
  "data": [
    "SHA256d",
    "Ethash"
  ]
}
```


# 用户模块
## 挖矿账户信息列表
获取用户下所有的挖矿账户信息。

**请求频率：** api-key级别每秒2次

#### HTTP请求
GET /v1/external/user/query

#### 请求示例
GET /v1/external/user/query

#### 返回值
参数 | 类型 | 含义
---|---|---
defaultAlgo | string | 创建挖矿账户时所选算法
realHashRate | string | 实时算力
hourHashRate | string | 小时算力
dayHashRate | string | 日算力
puid | Long | 挖矿账户id
pname | string | 挖矿账户
hashUnit | string | 算力单位(H)

#### 返回示例

```json
{
  "code": "200",
  "data": [
    {
      "dayHashRate": 200000000,
      "defaultAlgo": "SHA256d",
      "hashUnit": "H",
      "hourHashRate": 200000000,
      "pname": "test",
      "puid": 1,
      "realHashRate": 200000000
    }
  ],
  "msg": "success",
  "retry": false,
  "success": true
}
```

# 矿机模块
## 矿机列表
获取挖矿账户下所有矿机信息。

**请求频率：** api-key级别每秒2次

#### HTTP请求
GET /v1/external/worker/query

#### 请求示例
GET /v1/external/worker/query?algo=SHA256d&puid=4&sort=ASC&sortField=HASH_RATE&status=TOTAL

#### 请求参数

请求参数 | 类型 | 含义 | 是否必传
---|---|---|---
algo | String | 算法 | 是
puid | Long | 挖矿账户id | 是
sortField | String | 排序字段。WORKER:矿机名;HASH_RATE:实时算力;HASH_RATE_HOUR:小时算力;HASH_RATE_DAY:日算力;REJECT:拒绝率;LAST_SHARE_TIME:最后提交时间 | 是
sort | String | 排序方式。ASC:正序;DESC:倒序 | 是
status | String | 矿机类型。TOTAL:总数;ACTIVE:活跃;INACTIVE:不活跃;INVALID:无效 | 是

#### 返回值

参数 | 类型 | 含义
---|---|---
workerId | Long | 矿机id
workerName | string | 矿机名称
realHashRate | string | 实时算力
hourHashRate | string | 小时算力
dayHashRate | string | 日算力
hashUnit | string | 算力单位(H)
rejectRate | String | 拒绝率(百分比)
lastShareTime | Long | 最后提交时间

#### 返回示例

```json
{
  "code": "200",
  "data": {
    "currentPage": 1,
    "items": [
      {
        "dayHashRate": 10000000,
        "hashUnit": "H",
        "hourHashRate": 10000000,
        "lastShareTime": 10000000,
        "realHashRate": 10000000,
        "rejectRate": "0.0157",
        "workerId": 1,
        "workerName": "001"
      }
    ],
    "pageSize": 10,
    "totalNum": 100,
    "totalPage": 10
  },
  "msg": "success",
  "retry": false,
  "success": true
}
```


### 矿机明细
获取具体矿机信息。

**请求频率：** api-key级别每秒2次

#### HTTP请求
GET /v1/external/worker/get

#### 请求示例
GET /v1/external/worker/get?algo=SHA256d&currentPage=1&pageSize=10&puid=4&workerId=123

#### 请求参数

请求参数 | 类型 | 含义 | 是否必传
---|---|---|---
currentPage | Integer | 当前页 | 是
pageSize | Integer | 每页数量 | 是
algo | String | 算法 | 是
puid | Long | 挖矿账户id | 是
workerId | Long | 矿机id | 是

#### 返回值
参数 | 类型 | 含义
---|---|---
workerId | Long | 矿机id
workerName | string | 矿机名称
realHashRate | string | 实时算力
hourHashRate | string | 小时算力
dayHashRate | string | 日算力
hashUnit | string | 算力单位(H)
rejectRate | Long | 拒绝率(百分比)
lastShareTime | Long | 最后提交时间
status | String | 矿机类型。TOTAL:总数;ACTIVE:活跃;INACTIVE:不活跃;INVALID:无效

#### 返回示例

```json
{
  "code": "200",
  "data": {
    "dayHashRate": 10000000,
    "hashUnit": "H",
    "hourHashRate": 10000000,
    "lastShareTime": 10000000,
    "realHashRate": 10000000,
    "rejectRate": "0.0157",
    "workerId": 1,
    "workerName": "001"
    "status": "TOTAL"
  },
  "msg": "success",
  "retry": false,
  "success": true
}
```

# 收益模块
### 收益列表
获取挖矿账户的挖矿收益

**请求频率：** api-key级别每秒2次

#### HTTP请求
GET /v1/external/earn/query

#### 请求示例
GET /v1/external/earn/query?currentPage=1&pageSize=50&algo=SHA256d&puid=1&startTime=1646040033139&endTime=1646040040143

#### 请求参数

请求参数 | 类型 | 含义 | 是否必传
---|---|---|---
currentPage | Integer | 当前页 | 是
pageSize | Integer | 每页数量 | 是
algo | String | 算法 | 是
puid | Long | 挖矿账户id | 是
startTime | Long | 挖矿开始日期 时间戳(UTC) | 否
endTime | Long | 挖矿结束日期 时间戳(UTC) | 否

#### 返回值
参数 | 类型 | 含义
---|---|---
amount | string | 收益
currency | string | 收益单位
miningDate | Long | 挖矿日期
settleTime | Long | 打款时间
hashRate | string | 日算力
hashUnit | string | 算力单(H)
status | string | 支付状态。WAIT_SETTLE:待支付;SETTLED:已支付

#### 返回示例

```json
{
  "code": "200",
  "data": {
    "currentPage": 1,
    "items": [
      {
        "amount": 1.234,
        "currency": "BTC",
        "hashRate": 100000,
        "hashUnit": "H",
        "miningDate": 1646040033139,
        "settleTime": 1646040033139,
        "status": "WAIT_SETTLE"
      }
    ],
    "pageSize": 10,
    "totalNum": 100,
    "totalPage": 10
  },
  "msg": "success",
  "retry": false,
  "success": true
}
```

## 额外收益列表
获取挖矿账户的收益信息

**请求频率：** api-key级别每秒2次

#### HTTP请求
GET /v1/external/reward/query

#### 请求示例
GET /v1/external/reward/query?currentPage=1&pageSize=50&algo=SHA256d&puid=1&startTime=1646040033139&endTime=1646040040143

#### 请求参数

请求参数 | 类型 | 含义 | 是否必传
---|---|---|---
currentPage | Integer | 当前页 | 是
pageSize | Integer | 每页数量 | 是
algo | String | 算法 | 是
puid | Long | 挖矿账户id | 是
startTime | Long | 开始日期 时间戳(UTC) | 否
endTime | Long | 结束日期 时间戳(UTC) | 否

#### 返回值
请求参数 | 类型 | 含义
---|---|---
amount | string | 收益
currency | string | 收益单位
miningDate | Long | 挖矿日期
settleTime | Long | 打款时间
hashRate | string | 算力
hashUnit | string | 算力单位(H)
status | string | 支付状态。WAIT_SETTLE:待支付;SETTLED:已支付
type | string | 补款类型。MINING_EARN:挖矿收益;UNITE_COIN_EARN:联合挖矿;REWARD_EARN:活动奖励

#### 返回示例

```json
{
  "code": "200",
  "data": {
    "currentPage": 1,
    "items": [
      {
        "amount": 1,
        "currency": "BTC",
        "miningDate": 1646040033139,
        "settleTime": 1646040033139,
        "status": "WAIT_SETTLE",
        "type": "MINING_EARN"
      }
    ],
    "pageSize": 10,
    "totalNum": 100,
    "totalPage": 10
  },
  "msg": "success",
  "retry": true,
  "success": true
}
```