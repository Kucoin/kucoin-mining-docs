# Quick Start
## Introduction
The KuCoin Pool Developer Documentation is an API document written for users who wish to obtain data from the KuCoin Pool.

## Upcoming Changes

**14/07/2022:**
- Add Accumulated income of user interface

**27/07/2022:**
- Add Accumulated income of all sub-accounts interface

# REST API
## API Server
Base URL: https://www.kucoin.com/_api/miningpool

The request URL needs to be determined by BASE and specific endpoint combination.

## Endpoint of the Interface
Each interface has its own endpoint, described by field HTTP REQUEST in the docs.

For the GET METHOD API, the endpoint needs to contain the query parameters string.

For example, to obtain mining income data, the default endpoint is /v1/external/earn/query. If your request parameter is algo=SHA256d, then the endpoint will be /v1/external/earn/query?algo=SHA256d. Therefore your ultimate request URL should be https://www.kucoin.com/_api/miningpool/v1/external/earn/query?algo=SHA256d.

## Request
- The **POST** request should be application/json.
- The return type of requests is application/json.

All timestamp parameters are calculated in Unix timestamp milliseconds unless otherwise stated (e.g., 1544657947759).

### Request Parameter
For **GET** requests, the parameters should be spliced into the request URL (e.g., /v1/external/earn/query?algo=SHA256d).

For **POST** and **PUT** requests, the parameters should be spliced into the request body in JSON format (e.g., {"algo": "SHA256d"}). Note: Do not add spaces to the JSON string.

### Returned Error
The system will return an HTTP error code or a system error code. You can troubleshoot the cause of the error based on the returned parameter messages.

### HTTP Error Code
Code | Description
---|---
400 | Bad Request -- Invalid request format.
401 | Unauthorized -- Invalid Api-Key.
403 | Forbidden or Too Many Requests -- Requests are forbidden or have exceeded the request frequency limit.
404 | Not Found -- The specified resource could not be found.
405 | Method Not Allowed -- The method you requested the resource from is invalid.
415 | Content-Type: application/json -- The request type must be application/json.
500 | Internal Server Error -- Server error, please try again later.
503 | Service Unavailable -- The server is under maintenance, please try again later.

### ystem Error Code
Code | Description
---|---
200002 | Please try again later -- Requests have been sent too frequently. The interface has begun limiting requests.
400001 | Any of key, timestamp, passphrase, version is missing in your request parameters -- Missing parameters for signature verification in the request parameters.
400002 | Invalid timestamp -- The time difference between the request and the server is more than 5 seconds.
400003 | Key not exists -- The API-Key does not exist.
400004 | Invalid passphrase -- The passphrase is incorrect.
400100 | Parameter Error -- The request parameter is invalid.
500000 | Internal Server Error -- Server error, please try again later.

If the system returns an HTTP status code of 200 but the service fails, the system will report an error. You can troubleshoot errors based on the returned parameter messages.。

### API Return
The API return is in JSON format.

When the system returns HTTP status code 200 and system code 200000, this means the response was successful. The return results are as follows.

1. General Requests

```
{
  "success": true,
  "code": "200", //status code
  "msg": "success", //error message
  "retry": false,
  "data": "" //returned data (any type, representing the returned data)
}
```

2. Pagination Requests

```
{
  "code": "200", //status code
  "data": {      //returned data (any type, representing the returned data)
    "currentPage": 1, //current page
    "items": [], //array data
    "pageSize": 10, //results of each page
    "totalNum": 100, //total results
    "totalPage": 10 //total pages
  },
  "msg": "success", //error message
  "retry": false,
  "success": true
}
```


### Pagination
Pagination allows the fetching of results using the current page count and is highly suitable for fetching real-time data. For example, the */v1/external/worker/query* and */v1/external/earn/query* endpoints both return the results of the first page by default, with a total of 50 pieces of data. For more data, specify additional pagination based on the currently returned data and then make the request.

#### Request Parameters
Parameter | Default Value | Description
---|---|---
currentPage | 1 | Current page
pageSize | 50 | Number of results per page (min: 10, max: 500)

Example
GET /v1/external/worker/query?currentPage=1&pageSize=50



# API Verification
## Create Api-Key
Before making a request through the API, you must create an Api Key from the official website from KuCoin Pool > Settings > API Management. After creating an API Key, store the following three pieces of information in a safe place.

- Api-Key
- Api-Secret
- Api-Passphrase

The Api-Key and Api-Secret are randomly generated and provided by KuCoin, while Api-Passphrase is defined by you when creating the API. If the above information is lost, it cannot be recovered and you will need to re-apply for an Api-Key.

## Creation Request
The REST request parameters must contain the following:

- key: Api-Key(passed as a string)
- timestamp: the timestamp of the request
- passphrase:  the password of the API Key when creating the API
- version: the key version number, current version number convention=2


**passphrase:**in the request parameters

1. Use the API-Secret to encrypt the passphrase with **HMAC-sha256**, then pass the encrypted content through **base64** encoding.


```
#Example for get algo list in python

import base64
import hashlib
import hmac
import time
import requests

url = 'https://www.kucoin.com/_api/miningpool/v1/external/algo/query'
api_key = "api_key"
api_secret = "api_secret"
api_passphrase = "api_passphrase"
now = int(time.time() * 1000)
params = {
    'key': api_key,
    'passphrase': base64.b64encode(
        hmac.new(api_secret.encode('utf-8'), api_passphrase.encode('utf-8'), hashlib.sha256).digest()),
    'timestamp': now,
    'version': 2
}
response = requests.get(url, params=params)
print(response.status_code)
print(response.json())

```

# Glossary
- User: Official website login account
- Sub-accounts: mining accounts created by users in the mining pool system

# Algorithms
## Algorithm List
Get the algorithms supported for mining under KuCoin Mining Pool.

**Frequency of requests:** 2 times per second at Api-Key level.


#### HTTP请求
GET /v1/external/algo/query

#### Request Example
GET /v1/external/algo/query

#### Return Value
Parameter | Type | Description
---|---|---
data | list | Algorithm

### Return Example

```
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


# User
## Sub-account Information
Get information about all sub accounts of the user.

**Frequency of requests:** 2 times per second at Api-Key level.

#### HTTP Request
GET /v1/external/user/query

#### Request Example
GET /v1/external/user/query

#### Return Value
Parameter | Type | Description
---|---|---
defaultAlgo | string | Algorithm selected when creating a sub account
realHashRate | string | Real-time hashrate
hourHashRate | string | Hourly hashrate
dayHashRate | string | Daily hashrate
puid | Long | Sub account ID
pname | string | Sub account name
hashUnit | string | Hashrate unit (H)

#### Return Example

```
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

# Mining Machines
## List of Mining Machines
Get information about all mining machines under the sub account.

**Frequency of requests:** 2 times per second at Api-Key level.

#### HTTP Request
GET /v1/external/worker/query

#### Request Example
GET /v1/external/worker/query?algo=SHA256d&puid=4&sort=ASC&sortField=HASH_RATE&status=TOTAL

#### Request Parameters

Request Parameter | Type | Description | Required
---|---|---|---
currentPage | Integer | Current page | Yes
pageSize | Integer | Results of each page | Yes
algo | String | Algorithm | Yes
puid | Long | Sub account ID | Yes
sortField | String | Sort field WORKER: Name of the mining machine; HASH_RATE: Real-time hashrate; HASH_RATE_HOUR: Hourly hashrate; HASH_RATE_DAY: Daily hashrate; REJECT: Rejection rate; LAST_SHARE_TIME: Last submitted time | Yes
sort | String | Sort by ASC: ascending; DESC: descending | Yes
status | String | Type of mining machine TOTAL: Total; ACTIVE: Active; INACTIVE: Inactive; INVALID: Invalid | Yes

#### Return Value

Parameter | Type | Description
---|---|---
workerId | Long | Mining machine ID
workerName | string | Mining machine name
realHashRate | string | Real-time hashrate
hourHashRate | string | Hourly hashrate
dayHashRate | string | Daily hashrate
hashUnit | string | Hashrate unit (H)
rejectRate | String | Rejection rate (%)
lastShareTime | Long | Last submitted time

#### Return Example

```
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


### Mining Machine Details
Get detailed information about a mining machine.

**Frequency of requests:** 2 times per second at Api-Key level.

#### HTTP Request
GET /v1/external/worker/get

#### Request Example
GET /v1/external/worker/get?algo=SHA256d&currentPage=1&pageSize=10&puid=4&workerId=123

#### Request Parameters

Request Parameter | Type | Description | Required
---|---|---|---
currentPage | Integer | Current page | Yes
pageSize | Integer | Results of each page | Yes
algo | String | Algorithm | Yes
puid | Long | Sub account ID | Yes
workerId | Long | Mining machine ID | Yes

#### Return Value
Parameter | Type | Description
---|---|---
workerId | Long | Mining machine ID
workerName | string | Mining machine name
realHashRate | string | Real-time hashrate
hourHashRate | string | Hourly hashrate
dayHashRate | string | Daily hashrate
hashUnit | string | Hashrate unit (H)
rejectRate | Long | Rejection rate (%)
lastShareTime | Long | Last submitted time
status | String | Type of mining machine TOTAL: Total; ACTIVE: Active; INACTIVE: Inactive; INVALID: Invalid

#### Return Example

```
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
    "status": "ACTIVE"
  },
  "msg": "success",
  "retry": false,
  "success": true
}
```

# Income
### Income List
Get information about the mining income of the sub account.

**Frequency of requests：** 2 times per second at Api-Key level.

#### HTTP Request
GET /v1/external/earn/query

#### Request Example
GET /v1/external/earn/query?currentPage=1&pageSize=50&algo=SHA256d&puid=1&startTime=1646040033139&endTime=1646040040143

#### Request Parameters

Request Parameter | Type | Description | Required
---|---|---|---
currentPage | Integer | Current page | Yes
pageSize | Integer | Results of each page | Yes
algo | String | Algorithm | Yes
puid | Long | Sub account ID  | Yes
startTime | Long | Mining start date timestamp (UTC)  | No
endTime | Long | Mining end date timestamp (UTC) | No

#### Return Value
Parameter | Type | Description
---|---|---
amount | string | Income
currency | string | Unit of income
miningDate | Long | Mining date
settleTime | Long | ncome time
hashRate | string | Daily hashrate
hashUnit | string | Hashrate unit (H)
status | string | Payment status WAIT_SETTLE: Pending; SETTLED: Paid

#### Return Example

```
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

## Extra Income
Get information about the income of the sub account.

**requency of requests:** 2 times per second at Api-Key level.

#### HTTP Request
GET /v1/external/reward/query

#### Request Example
GET /v1/external/reward/query?currentPage=1&pageSize=50&algo=SHA256d&puid=1&startTime=1646040033139&endTime=1646040040143

#### Request Parameters

Request Parameter | Type | Description | Required
---|---|---|---
currentPage | Integer | Current page | Yes
pageSize | Integer | Results of each page  | Yes
algo | String | Algorithm | Yes
puid | Long | Sub account ID | Yes
startTime | Long | Start date timestamp (UTC) | No
endTime | Long | End date timestamp (UTC)  | No

#### Return Value
Request Parameter | Type | Description
---|---|---
amount | string | Income
currency | string | Unit of income
miningDate | Long | Mining date
settleTime | Long | Income time
hashRate | string | Hashrate
hashUnit | string | Hashrate unit (H)
status | string | Payment status WAIT_SETTLE: Pending; SETTLED: Paid
type | string | Type of income MINING_EARN: Mining income; UNITE_COIN_EARN: Merged mining; REWARD_EARN: Event rewards; HASH_RATE_EARN:Retroactive Mining Income; FEE_RATE_EARN:Mining Fee Subsidies

#### Return Example

```
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

## Accumulated income of user
get accumulated income of user.

**requency of requests:** 2 times per second at Api-Key level.

#### HTTP Request
GET /v1/external/user/earn

#### Request Example
GET /v1/external/user/earn?coinName=BTC

#### Request Parameters

Request Parameter | Type | Description | Required
---|---|---|---
coinName | String | currency | Yes

#### Return Example

```json
{
  "success": true,
  "code": "200",
  "msg": "success",
  "retry": false,
  "data": 12   //Currency total income
}
```

## Accumulated income of all sub-accounts
get accumulated income of all sub-accounts.

**requency of requests:** 2 times per second at Api-Key level.

#### HTTP Request
GET /v1/external/sub-user/earn

#### Request Example
GET /v1/external/sub-user/earn?coinName=BTC

#### Request Parameters

Request Parameter | Type | Description | Required
---|---|---|---
coinName | String | currency | Yes

#### Return Value
Request Parameter | Type | Description
---|---|---
puid | Long | Sub account ID
pname | string | Sub account name
amount | string | Income

#### Return Example

```json
{
  "code": "200",
  "data": [
    {
      "puid": 1,
      "pname": "test",
      "amount": 0.1
    }
  ],
  "msg": "success",
  "retry": false,
  "success": true
}
```