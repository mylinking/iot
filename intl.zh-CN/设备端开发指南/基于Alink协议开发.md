# 基于Alink协议开发 {#concept_lgm_24w_5db .concept}

物联网平台提供设备端SDK，已经将设备端与云端交互的协议封装好，您可以直接使用设备端SDK来开发。然而，由于嵌入式环境复杂，若已提供的设备端SDK不能满足您的需求，您可以参考本文，自行封装Alink协议数据，建立设备与云端的通信。ALink协议是针对物联网开发领域设计的一种数据交换规范，数据格式是JSON，用于设备端和云端的双向通信，更便捷地实现和规范了设备端和云端之间的业务数据交互。

## 设备上线核心流程 {#section_sp3_4vg_12b .section}

如下图所示，设备上线流程，可以按照设备类型，分为直连设备接入与子设备接入。主要包括：设备注册、上线和数据上报三个流程。

直连设备接入有两种方式：

-   使用[一机一密](../../../../intl.zh-CN/设备端开发指南/设备安全认证/一机一密.md#)方式提前烧录三元组，注册设备，上线，然后上报数据。
-   使用[一型一密](../../../../intl.zh-CN/设备端开发指南/设备安全认证/一型一密.md#)动态注册提前烧录产品证书（ProductKey和ProductSecret），注册设备， 上线，然后上报数据。

子设备接入流程通过网关发起，具体接入方式有两种：

-   使用[一机一密](../../../../intl.zh-CN/设备端开发指南/设备安全认证/一机一密.md#)提前烧录三元组，子设备上报三元组给网关，网关添加拓扑关系，复用网关的通道上报数据。
-   使用动态注册方式提前烧录ProductKey，子设备上报ProductKey和DeviceName给网关，云端校验DeviceName成功后，下发DeviceSecret。子设备将获得的三元组信息上报网关，网关添加拓扑关系，通过网关的通道上报数据。

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/7638/15362133955802_zh-CN.png)

## 一、设备身份注册 {#section_gm3_svg_12b .section}

接入IoT平台的设备身份注册有两种方式：

-   使用一机一密的方式，在控制台申请三元组（ProductKey， DeviceName， DeviceSecret）做为设备唯一标识，您只需要将申请到的三元组预烧录到固件，固件在完成上线建连后即可向云端上报数据。
-   使用动态注册的方式，包括直连设备使用一型一密动态注册和子设备动态注册。
    -   直连设备使用一型一密动态注册的流程：
        1.  在控制台预注册，并获取产品证书（ProductKey和ProductSecret）。其中，预注册时DeviceName使用可以直接从设备中读出的数据，如mac地址或SN序列号等。
        2.  在控制台开启产品的动态注册开关。
        3.  将产品证书烧录至固件。
        4.  设备直接向云端发起身份认证。云端认证成功后，下发DeviceSecret。
        5.  设备使用三元组与云端建立连接。
    -   子设备动态注册流程：
        1.  在控制台预注册获取ProductKey，其中，预注册时DeviceName使用可以直接从设备中读出的数据，如mac地址或SN序列号等。
        2.  在控制台开启动态注册开关。
        3.  将子设备ProductKey烧录至固件或网关上。
        4.  网关代替子设备向云端发起身份注册。

**子设备的动态注册**

上行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/sub/register
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/sub/register\_reply

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ],
  "method": "thing.sub.register"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": [
    {
      "iotId": "12344",
      "productKey": "1234556554",
      "deviceName": "deviceName1234",
      "deviceSecret": "xxxxxx"
    }
  ]
}
```

参数说明

|参数|类型|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|List|设备动态注册的参数|
|deviceName|String|子设备的名称|
|productKey|String|子设备的产品Key|
|iotId|String|设备的唯一标识Id|
|deviceSecret|String|设备秘钥|
|method|String|请求方法|
|code|Integer|结果信息|

错误信息

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|6402|topo relation cannot add by self|设备不能将自己添加为自己的子设备|
|401|request auth error|签名校验失败|

**直连设备使用一型一密动态注册**

直连设备动态注册，通过HTTP请求进行。 使用时需在控制台上开通该产品的一型一密动态注册功能。

-   URL TMPLATE： [https://iot-auth.cn-shanghai.aliyuncs.com/auth/register/device](https://iot-auth.cn-shanghai.aliyuncs.com/auth/register/device)
-   HTTP METHOD： POST

Alink请求数据格式

```
POST /auth/register/device  HTTP/1.1
Host: iot-auth.cn-shanghai.aliyuncs.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 123
productKey=1234556554&deviceName=deviceName1234&random=567345&sign=adfv123hdfdh&signMethod=HmacMD5
```

Alink响应数据格式

```
{
  "code": 200,
  "data": {
    "productKey": "1234556554",
    "deviceName": "deviceName1234",
    "deviceSecret": "adsfweafdsf"
  },
  "message": "success"
}
```

参数说明

|参数|类型|说明|
|:-|:-|:-|
|productKey|String|产品唯一标识|
|deviceName|String|设备名称|
|random|String|随机数|
|sign|String|签名|
|signMethod|String|签名方法， 目前支持hmacmd5、hmacsha1、hmacsha256|
|code|Integer|结果信息|
|deviceSecret|String|设备秘钥|

签名方法

加签内容为将所有提交给服务器的参数（sign,signMethod除外）按照字母顺序排序, 然后将参数和值依次拼接（无拼接符号）。 然后对加签内容，使用signMethod指定的加签算法进行加签。`sign = hmac_sha1(productSecret, deviceNamedeviceName1234productKey1234556554random123)`

## 二、添加拓扑关系 {#section_w33_vyg_12b .section}

子设备身份注册后，下一步需由网关向云端上报[网关与子设备](../../../../intl.zh-CN/用户指南/产品与设备/网关与子设备.md#)的拓扑关系，然后进行上线。上线过程中，云端会校验子设备的身份和与拓扑关系。验证合法，才会建立并绑定子设备逻辑通道至网关物理通道上。子设备与云端的数据上下行与直连设备协议可以完全保持一致，协议上不需要露出网关信息。

删除拓扑关系后，子设备利用该网关再次上线时，系统将提示拓扑关系不存在，认证不通过。

**添加物的拓扑关系**

上行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/topo/add
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/topo/add\_reply

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554",
      "sign": "xxxxxx",
      "signmethod": "hmacSha1",
      "timestamp": "1524448722000",
      "clientId": "xxxxxx"
    }
  ],
  "method": "thing.topo.add"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|类型|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|List|请求入参|
|deviceName|String|设备的名称，在request取子设备名称|
|productKey|String|设备的产品Key， 在request取子设备名称|
|sign|String|签名|
|signmethod|String|签名方法，支持hmacSha1,hmacSha256,hmacMd5, Sha256|
|timestamp|String|时间戳|
|clientId|String|本地标记，非必填，可以和productKey&deviceName保持一致|
|code|Integer|结果信息， 200表示成功|

加签算法

**说明：** 物联网平台的加签算法都是通用的。

1.  加签内容为将所有提交给服务器的参数（sign,signMethod除外）按照字母顺序排序, 然后将参数和值依次拼接（无拼接符号）。 然后对加签内容，使用signMethod指定的加签算法进行加签。
2.  如按照示例中rerequest请求，对request请求中的params中的参数进行加签.
3.  `sign= hmac_md5(deviceSecret, clientId123deviceNametestproductKey123timestamp1524448722000)`

错误信息

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|6402|topo relation cannot add by self|设备不能把自己添加为自己的子设备|
|401|request auth error|签名校验授权失败|

**删除物的拓扑关系**

网关类型的设备， 可以通过该Topic上行请求删除它和子设备之间的拓扑关系。

上行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/topo/delete
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/topo/delete\_reply

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ],
  "method": "thing.topo.delete"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|类型|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|List|请求参数|
|deviceName|String|设备名称， 在request中为子设备的名称|
|productKey|String|设备产品Key，在request中为子设备产品的Key|
|method|String|请求方法|
|code|Integer|结果信息， 200表示成功|

错误信息

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|6100|device not found|设备不存在|

**获取物的拓扑关系**

上行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/topo/get
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/topo/get\_reply

网关类型的设备，可以通过该Topic获取该设备和关联的子设备拓扑关系。

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": {},
  "method": "thing.topo.get"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ]
}
```

参数说明

|参数|类型|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|请求参数，可为空|
|method|String|请求方法|
|deviceName|String|子设备的名称|
|productKey|String|子设备的产品Key|
|code|Integer|结果信息， 200表示成功|

错误信息

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|

**发现设备列表上报**

上行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/list/found
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/list/found\_reply。在一些场景下，网关可以发现新接入的子设备。发现后，需将新接入子设备的信息上报云端，然后通过数据流转到第三方应用，选择将哪些子设备接入该网关。

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ],
  "method": "thing.list.found"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data":{}
}
```

参数说明

|参数|类型|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|请求参数，可为空|
|method|String|请求方法|
|deviceName|String|子设备的名称|
|productKey|String|子设备的产品Key|
|code|Integer|结果信息， 200表示成功|

错误信息

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|6250|product not found|上报的子设备产品不存在|
|6280|devicename not meet specs|上报的子设备的名称不符规范，设备名称支持英文字母、数字和特殊字符-\_@.:，长度限制4~32|

**通知添加设备拓扑关系**

下行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/topo/add/notify
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/topo/add/notify\_reply

通知网关设备对子设备发起添加拓扑关系，可以配合发现设备列表上报功能使用。可以通过数据流转获取设备返回的结果，数据流转topic为`/{productKey}/{deviceName}/thing/downlink/reply/message`.

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": [
    {
      "deviceName": "deviceName1234",
      "productKey": "1234556554"
    }
  ],
  "method": "thing.topo.add.notify"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|类型|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|请求参数，可为空|
|method|String|请求方法|
|deviceName|String|子设备的名称|
|productKey|String|子设备的产品Key|
|code|Integer|结果信息， 200表示成功|

## 三、设备上线 {#section_iqk_mzg_12b .section}

直连设备上线之前仅需确保已在云端注册身份。

子设备上线需要确保身份已经注册、已添加拓扑关系。因为子设备上线，云端需要根据拓扑关系进行身份校验，以确定子设备是否具备复用网关通道的能力。

**子设备上线**

上行

-   TOPIC: /ext/session/\{productKey\}/\{deviceName\}/combine/login
-   REPLY TOPIC: /ext/session/\{productKey\}/\{deviceName\}/combine/login\_reply

Alink请求数据格式

```
{
  "id": "123",
  "params": {
    "productKey": "123",
    "deviceName": "test",
    "clientId": "123",
    "timestamp": "123",
    "signMethod": "hmacmd5",
    "sign": "xxxxxx",
    "cleanSession": "true"
  }
}
```

Alink响应数据格式

```
{
  "id":"123",
  "code":200,
  "message":"success"
  "data":""
}
```

签名方法

1.  加签内容为将所有提交给服务器的参数（sign,signMethod除外）按照字母顺序排序, 然后将参数和值依次拼接（无拼接符号）。 然后对加签内容，使用signMethod指定的加签算法进行加签。
2.  `sign= hmac_md5(deviceSecret, cleanSessiontrueclientId123deviceNametestproductKey123timestamp123)`

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|params|List|请求入参|
|deviceName|String|子设备的设备名称|
|productKey|String|子设备所属的产品Key|
|sign|String|子设备签名，规则与网关相同|
|signmethod|String|签名方法，支持hmacSha1,hmacSha256,hmacMd5, Sha256|
|timestamp|String|时间戳|
|clientId|String|设备端标识，可以写productKey&deviceName|
|cleanSession|String|如果是true，那么清理所有子设备离线消息，即QoS1的所有未接收内容|
|code|Integer|结果信息， 200表示成功|
|message|String|结果信息|
|data|String|结果中附加信息，json格式|

**说明：** 网关下同时在线的子设备数目不能超过200，超过后，新的子设备上线请求将被拒绝。

错误信息

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|429|rate limit, too many subDeviceOnline msg in one minute|单个设备认证过于频繁被限流|
|428|too many subdevices under gateway|网关下同时上线子设备过多|
|6401|topo relation not exist|网关和子设备没有拓扑关系|
|6100|device not found|子设备不存在|
|521|device deleted|子设备被删除|
|522|device forbidden|子设备被禁用|
|6287|invalid sign|子设备密码或者签名错误|

**子设备下线**

上行

-   TOPIC: /ext/session/\{productKey\}/\{deviceName\}/combine/logout
-   REPLY TOPIC: /ext/session/\{productKey\}/\{deviceName\}/combine/logout\_reply

Alink请求数据格式

```
{
  "id": 123,
  "params": {
    "productKey": "xxxxx",
    "deviceName": "xxxxx"
  }
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "message": "success",
  "data": ""
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|params|List|请求入参|
|deviceName|String|子设备的设备名称|
|productKey|String|子设备所属的产品Key|
|code|Integer|结果信息， 200表示成功|
|message|String|结果信息|
|data|String|结果中附加信息，json格式|

错误信息

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|520|device no session|子设备会话不存在|

子设备接入具体可参考[子设备接入](../../../../intl.zh-CN/设备端开发指南/子设备接入/子设备接入.md#)，错误码参考[错误码说明](../../../../intl.zh-CN/设备端开发指南/子设备接入/错误码说明.md#)。

## 四、设备属性、事件、服务协议 {#section_g4j_5zg_12b .section}

设备的数据上报方式分为标准模式和透传模式两种方式，两者二选一不能混用。

1.  透传模式是指设备上报原始数据如二进制数据流，阿里云IoT平台会运行客户提交的脚本将原始数据转成标准数据格式。 脚本的使用请参考[脚本解析](https://help.aliyun.com/document_detail/68702.html)。
2.  标准模式是指设备按照标准的数据格式生成数据，然后上报数据。具体格式可以参考该文档中设备数据传输的request请求和response响应。

**设备属性上报**

**说明：** 参数按照TSL中出入参定义。

上行（透传）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/model/up\_raw
-   REPLY TOPIC：/sys/\{productKey\}/\{deviceName\}/thing/model/up\_raw\_reply

上行（非透传）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/event/property/post
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/event/property/post\_reply

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": {
    "Power": {
      "value": "on",
      "time": 1524448722000
    },
    "WF": {
      "value": 23.6,
      "time": 1524448722000
    }
  },
  "method": "thing.event.property.post"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|请求参数|
|method|String|请求方法|
|Power|String|属性名称|
|value|String|属性的值|
|time|Long|时间戳， 类型为utc毫秒数|
|code|Integer|结果信息|

错误信息

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|6106|map size must less than 200|设备上报属性一次性最多只能上报200条属性|
|6313|tsl service not available|用户上报属性时会进行校验，检查上报的属性是否符合用户定义的属性格式，当校验服务不可用时会报这个错。 属性校验参考[什么是物模型](../../../../intl.zh-CN/用户指南/产品与设备/高级版/什么是物模型.md#)|

**说明：** 云端会校验上报的属性信息，通过产品TSL描述判断上传的属性是否符合您定义的属性格式。不合格属性将直接过滤，仅保留合格属性。若所有属性都不合格，校验会过滤掉全部属性，返回的response也是成功的。

**设备属性设置**

**说明：** 参数按照TSL中出入参定义。

下行（透传）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/model/down\_raw
-   REPLY TOPIC /sys/\{productKey\}/\{deviceName\}/thing/model/down\_raw\_reply

下行（非透传）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/service/property/set
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/service/property/set\_reply

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": {
    "temperature": "30.5"
  },
  "method": "thing.service.property.set"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|属性设置参数|
|method|String|请求方法|
|temperature|String|属性名称|
|code|Integer|结果信息， 具体参考设备端通用code|

**设备属性获取**

**说明：** 

-   参数按照TSL中出入参定义。
-   目前物的属性统一从云端获取，而非实时从设备获取。

下行（透传）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/model/down\_raw
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/model/down\_raw\_reply

设备收到获取设备属性指令之后，通过reply消息向云端返回获取的属性信息。可以通过数据流转获取返回的属性信息， 数据流转Topic为`{productKey}/{deviceName}/thing/downlink/reply/message`。

payload: 0x001FFEE23333

下行（非透传）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/service/property/get
-   REPLY TOPIC：/sys/\{productKey\}/\{deviceName\}/thing/service/property/get\_reply

设备收到下行获取设备属性指令之后，通过reply消息向云端返回获取的属性信息。云端可以通过数据流转获取返回的属性信息，数据流转Topic为`/{productKey}/{deviceName}/thing/downlink/reply/message`。

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": [
    "power",
    "temp"
  ],
  "method": "thing.service.property.get"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {
    "power": "on",
    "temp": "23"
  }
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|List|需要获取的属性名称列表|
|method|String|请求方法|
|power|String|属性名称|
|temp|String|属性名称|
|code|Integer|结果信息|

**设备事件上报**

上行（透传）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/model/up\_raw
-   REPLY TOPIC：/sys/\{productKey\}/\{deviceName\}/thing/model/up\_raw\_reply

上行（非透传）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/event/\{tsl.event.identifier\}/post
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/event/\{tsl.event.identifier\}/post\_reply

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": {
    "value": {
      "Power": "on",
      "WF": "2"
    },
    "time": 1524448722000
  },
  "method": "thing.event.{tsl.event.identifier}.post"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|List|事件上报参数|
|method|String|请求方法|
|value|Object|事件参数的值|
|Power|String|事件参数名称， 根据TSL模板选择|
|WF|String|事件参数名称， 根据TSL模板选择|
|code|Integer|结果信息，具体参考设备端通用code|
|time|Long|生成的时间戳，类型为utc毫秒|

**说明：** 

-   tsl.event.identifier 为TSL模板中事件的描述符。TSL模板具体参考[什么是物模型](../../../../intl.zh-CN/用户指南/产品与设备/高级版/什么是物模型.md#)。
-   对于上报的事件云端会做校验，通过产品的TSL描述判断上报的事件是否符合用户定义的事件格式。不合格的事件会直接过滤掉，并返回失败的信息。

**设备服务触发**

透传（下行）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/model/down\_raw
-   REPLY TOPIC : /sys/\{productKey\}/\{deviceName\}/thing/model/down\_raw\_reply

如果设备服务调用方式选择同步方式（物联网平台控制台中提前配置，产品服务定义中选择），云端直接使用RRPC同步方式下行推送，设备RRPC的集成方式，详见[文档介绍](../../../../intl.zh-CN/最佳实践/远程控制设备并返回结果.md#)；

如果设备端服务调用方式选择异步方式（物联网平台控制台中提前配置，产品服务定义中选择），云端则采用异步方式下行推送，设备也采用异步方式返回。只有当前服务选择了异步方式，云端才会订阅该异步reply Topic：/sys/\{productKey\}/\{deviceName\}/thing/model/down\_raw\_reply。 异步调用的结果，可以通过数据流转获取，数据流转Topic为`/{productKey}/{deviceName}/thing/downlink/reply/message`。

非透传（下行）

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/service/\{tsl.service.identifier\}
-   REPLY TOPIC：/sys/\{productKey\}/\{deviceName\}/thing/service/\{tsl.service.identifier\}\_reply；

如果设备服务调用方式选择同步方式（物联网平台控制台产品管理，产品服务定义中选择），云端直接使用RRPC同步方式下行推送，设备RRPC的集成方式，详见[文档介绍](../../../../intl.zh-CN/最佳实践/远程控制设备并返回结果.md#)；

如果设备端服务调用方式选择异步方式（物联网平台控制台产品管理，产品服务定义中选择），云端则采用异步方式下行推送，设备也采用异步方式返回，。只有当前服务选择了异步方式，云端才会订阅该异步reply topic：/sys/\{productKey\}/\{deviceName\}/thing/service/\{tsl.service.identifier\}\_reply。 异步调用的结果，可以通过数据流转获取， 数据流转topic为`/{productKey}/{deviceName}/thing/downlink/reply/message`。

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": {
    "Power": "on",
    "WF": "2"
  },
  "method": "thing.service.{tsl.service.identifier}"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": { }
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|List|服务调用参数|
|method|String|请求方法|
|value|Object|事件参数名称|
|Power|String|事件参数名称|
|WF|String|事件参数名称|
|code|Integer|结果信息，具体参考设备端通用code|

**说明：** tsl.service.identifier 为tsl模板中定义的服务描述符。TSL使用参考[什么是物模型](../../../../intl.zh-CN/用户指南/产品与设备/高级版/什么是物模型.md#)。

## 网关配置下发 {#section_lmz_ykc_z2b .section}

物模型扩展配置及子设备的连接协议配置，由云端推送至设备端。

-   TOPIC：/sys/\{productKey\}/\{deviceName\}/thing/model/config/push

Alink配置推送数据格式

```
{
  "id": 123,
  "version": "1.0",
  "method": "thing.model.config.push",
  "data": {
    "digest":"",
    "digestMethod":"",
    "url": ""
  }
}
```

参数说明

|参数|类型|说明|
|--|--|--|
|id|String|消息ID号|
|version|String|协议版本号，默认为1.0|
|method|String|方法，使用`thing.model.config.push`|
|data|Object|数据|
|digest|String|签名，用于校验从url中获取的数据完整性|
|digestMethod|String|签名方法，默认Sha256|
|url|String|数据url，从oss中请求获取|

Alink响应数据格式

```
{
  "id":123,
  "code":200,
  "message":"success",
  "data":{
    "digest":"",
    "digestMethod":"",
    "url":""
  }
}
```

url中的数据格式

```
{
  "modelList": [
    {
      "profile": {
        "productKey": "test01"
      },
      "services": [
        {
          "outputData": "",
          "identifier": "AngleSelfAdaption",
          "inputData": [
            {
              "identifier": "test01",
              "index": 0
            }
          ],
          "displayName": "test01"
        }
      ],
      "properties": [
        {
          "identifier": "identifier",
          "displayName": "test02"
        },
        {
          "identifier": "identifier_01",
          "displayName": "identifier_01"
        }
      ],
      "events": [
        {
          "outputData": [
            {
              "identifier": "test01",
              "index": 0
            }
          ],
          "identifier": "event1",
          "displayName": "abc"
        }
      ]
    },
    {
      "profile": {
        "productKey": "test02"
      },
      "properties": [
        {
          "originalDataType": {
            "specs": {
              "registerCount": 1,
              "reverseRegister": 0,
              "swap16": 0
            },
            "type": "bool"
          },
          "identifier": "test01",
          "registerAddress": "0x03",
          "scaling": 1,
          "operateType": "inputStatus",
          "pollingTime": 1000,
          "trigger": 1
        },
        {
          "originalDataType": {
            "specs": {
              "registerCount": 1,
              "reverseRegister": 0,
              "swap16": 0
            },
            "type": "bool"
          },
          "identifier": "test02",
          "registerAddress": "0x05",
          "scaling": 1,
          "operateType": "coilStatus",
          "pollingTime": 1000,
          "trigger": 2
        }
      ]
    }
  ],
  "serverList": [
    {
      "baudRate": 1200,
      "protocol": "RTU",
      "byteSize": 8,
      "stopBits": 2,
      "parity": 1,
      "name": "modbus01",
      "serialPort": "0",
      "serverId": "D73251B4277742"
    },
    {
      "protocol": "TCP",
      "port": 8000,
      "ip": "192.168.0.1",
      "name": "modbus02",
      "serverId": "586CB066D6A34"
    },
    {
      "password": "XIJTginONohPEUAyZxLB7Q==",
      "secPolicy": "Basic128Rsa15",
      "name": "server_01",
      "secMode": "Sign",
      "userName": "123",
      "serverId": "55A9D276A7ED470",
      "url": "tcp:00",
      "timeout": 10
    },
    {
      "password": "hAaX5s13gwX2JwyvUkOAfQ==",
      "name": "service_09",
      "secMode": "None",
      "userName": "1234",
      "serverId": "44895C63E3FF401",
      "url": "tcp:00",
      "timeout": 10
    }
  ],
  "deviceList": [
    {
      "deviceConfig": {
        "displayNamePath": "123",
        "serverId": "44895C63E3FF4013924CEF31519ABE7B"
      },
      "productKey": "test01",
      "deviceName": "test_02"
    },
    {
      "deviceConfig": {
        "displayNamePath": "1",
        "serverId": "55A9D276A7ED47"
      },
      "productKey": "test01",
      "deviceName": "test_03"
    },
    {
      "deviceConfig": {
        "slaveId": 1,
        "serverId": "D73251B4277742D"
      },
      "productKey": "test02",
      "deviceName": "test01"
    },
    {
      "deviceConfig": {
        "slaveId": 2,
        "serverId": "586CB066D6A34E"
      },
      "productKey": "test02",
      "deviceName": "test02"
    }
  ],
  "tslList": [
    {
      "schema": "https://iotx-tsl.oss-ap-southeast-1.aliyuncs.com/schema.json",
      "profile": {
        "productKey": "test02"
      },
      "services": [
        {
          "outputData": [],
          "identifier": "set",
          "inputData": [
            {
              "identifier": "test02",
              "dataType": {
                "specs": {
                  "unit": "mm",
                  "min": "0",
                  "max": "1"
                },
                "type": "int"
              },
              "name": "测试功能02"
            }
          ],
          "method": "thing.service.property.set",
          "name": "set",
          "required": true,
          "callType": "async",
          "desc": "属性设置"
        },
        {
          "outputData": [
            {
              "identifier": "test01",
              "dataType": {
                "specs": {
                  "unit": "m",
                  "min": "0",
                  "max": "1"
                },
                "type": "int"
              },
              "name": "测试功能01"
            },
            {
              "identifier": "test02",
              "dataType": {
                "specs": {
                  "unit": "mm",
                  "min": "0",
                  "max": "1"
                },
                "type": "int"
              },
              "name": "测试功能02"
            }
          ],
          "identifier": "get",
          "inputData": [
            "test01",
            "test02"
          ],
          "method": "thing.service.property.get",
          "name": "get",
          "required": true,
          "callType": "async",
          "desc": "属性获取"
        }
      ],
      "properties": [
        {
          "identifier": "test01",
          "dataType": {
            "specs": {
              "unit": "m",
              "min": "0",
              "max": "1"
            },
            "type": "int"
          },
          "name": "测试功能01",
          "accessMode": "r",
          "required": false
        },
        {
          "identifier": "test02",
          "dataType": {
            "specs": {
              "unit": "mm",
              "min": "0",
              "max": "1"
            },
            "type": "int"
          },
          "name": "测试功能02",
          "accessMode": "rw",
          "required": false
        }
      ],
      "events": [
        {
          "outputData": [
            {
              "identifier": "test01",
              "dataType": {
                "specs": {
                  "unit": "m",
                  "min": "0",
                  "max": "1"
                },
                "type": "int"
              },
              "name": "测试功能01"
            },
            {
              "identifier": "test02",
              "dataType": {
                "specs": {
                  "unit": "mm",
                  "min": "0",
                  "max": "1"
                },
                "type": "int"
              },
              "name": "测试功能02"
            }
          ],
          "identifier": "post",
          "method": "thing.event.property.post",
          "name": "post",
          "type": "info",
          "required": true,
          "desc": "属性上报"
        }
      ]
    },
    {
      "schema": "https://iotx-tsl.oss-ap-southeast-1.aliyuncs.com/schema.json",
      "profile": {
        "productKey": "test01"
      },
      "services": [
        {
          "outputData": [],
          "identifier": "set",
          "inputData": [
            {
              "identifier": "identifier",
              "dataType": {
                "specs": {
                  "length": "2048"
                },
                "type": "text"
              },
              "name": "7614"
            },
            {
              "identifier": "identifier_01",
              "dataType": {
                "specs": {
                  "length": "2048"
                },
                "type": "text"
              },
              "name": "测试功能1"
            }
          ],
          "method": "thing.service.property.set",
          "name": "set",
          "required": true,
          "callType": "async",
          "desc": "属性设置"
        },
        {
          "outputData": [
            {
              "identifier": "identifier",
              "dataType": {
                "specs": {
                  "length": "2048"
                },
                "type": "text"
              },
              "name": "7614"
            },
            {
              "identifier": "identifier_01",
              "dataType": {
                "specs": {
                  "length": "2048"
                },
                "type": "text"
              },
              "name": "测试功能1"
            }
          ],
          "identifier": "get",
          "inputData": [
            "identifier",
            "identifier_01"
          ],
          "method": "thing.service.property.get",
          "name": "get",
          "required": true,
          "callType": "async",
          "desc": "属性获取"
        },
        {
          "outputData": [],
          "identifier": "AngleSelfAdaption",
          "inputData": [
            {
              "identifier": "test01",
              "dataType": {
                "specs": {
                  "min": "1",
                  "max": "10",
                  "step": "1"
                },
                "type": "int"
              },
              "name": "参数1"
            }
          ],
          "method": "thing.service.AngleSelfAdaption",
          "name": "角度自适应校准",
          "required": false,
          "callType": "async"
        }
      ],
      "properties": [
        {
          "identifier": "identifier",
          "dataType": {
            "specs": {
              "length": "2048"
            },
            "type": "text"
          },
          "name": "7614",
          "accessMode": "rw",
          "required": true
        },
        {
          "identifier": "identifier_01",
          "dataType": {
            "specs": {
              "length": "2048"
            },
            "type": "text"
          },
          "name": "测试功能1",
          "accessMode": "rw",
          "required": false
        }
      ],
      "events": [
        {
          "outputData": [
            {
              "identifier": "identifier",
              "dataType": {
                "specs": {
                  "length": "2048"
                },
                "type": "text"
              },
              "name": "7614"
            },
            {
              "identifier": "identifier_01",
              "dataType": {
                "specs": {
                  "length": "2048"
                },
                "type": "text"
              },
              "name": "测试功能1"
            }
          ],
          "identifier": "post",
          "method": "thing.event.property.post",
          "name": "post",
          "type": "info",
          "required": true,
          "desc": "属性上报"
        },
        {
          "outputData": [
            {
              "identifier": "test01",
              "dataType": {
                "specs": {
                  "min": "1",
                  "max": "20",
                  "step": "1"
                },
                "type": "int"
              },
              "name": "测试参数1"
            }
          ],
          "identifier": "event1",
          "method": "thing.event.event1.post",
          "name": "event1",
          "type": "info",
          "required": false
        }
      ]
    }
  ]
}
```

参数说明如下：

|参数|类型|说明|
|--|--|--|
|modelList|Object|网关下面所有子设备的产品扩展信息|
|serverList|Object|网关下面所有的子设备通道管理|
|deviceList|Object|网关下面所有子设备的连接配置|
|tslList|Object|网关下面所有子设备的TSL描述|

-   modelList说明

    设备协议目前支持Modbus和OPC UA两种协议，两种协议的扩展信息不一致。

    -   Modbus

        ```
        {
          "profile": {
            "productKey": "test02"
          },
          "properties": [
            {
              "originalDataType": {
                "specs": {
                  "registerCount": 1,
                  "reverseRegister": 0,
                  "swap16": 0
                },
                "type": "bool"
              },
              "identifier": "test01",
              "registerAddress": "0x03",
              "scaling": 1,
              "operateType": "inputStatus",
              "pollingTime": 1000,
              "trigger": 1
            },
            {
              "originalDataType": {
                "specs": {
                  "registerCount": 1,
                  "reverseRegister": 0,
                  "swap16": 0
                },
                "type": "bool"
              },
              "identifier": "test02",
              "registerAddress": "0x05",
              "scaling": 1,
              "operateType": "coilStatus",
              "pollingTime": 1000,
              "trigger": 2
            }
          ]
        }
        ```

        参数说明如下：

        |参数|类型|说明|
        |--|--|--|
        |identifier|String|属性、事件和服务的描述符|
        |operateType|String|参数类型。取值可以为        -   线圈状态：coilStatus
        -   输入状态：inputStatus
        -   保持寄存器：holdingRegister
        -   输入寄存器：inputRegister
|
        |registerAddress|String|寄存器地址|
        |originalDataType|Object|数据原始类型|
        |type|String|取值。int16，uint16, int32，uint32，int64，uint64， float，double, string，customized data|
        |specs|Object|描述信息|
        |registerCount|Integer|寄存器的数据个数|
        |swap16|Integer|把寄存器内16位数据的前后8个bits互换。0 false, 1 true。|
        |reverseRegiste|Integer|把原始数据32位数据的bits互换。0 false, 1 true。|
        |scaling|Integer|缩放因子|
        |pollingTime|Integer|采集间隔|
        |trigger|Integer|数据的上报方式。1 按时上报，2 变更上报。|

    -   OPC UA

        ```
        {
          "profile": {
            "productKey": "test01"
          },
          "services": [
            {
              "outputData": "",
              "identifier": "AngleSelfAdaption",
              "inputData": [
                {
                  "identifier": "test01",
                  "index": 0
                }
              ],
              "displayName": "test01"
            }
          ],
          "properties": [
            {
              "identifier": "identifier",
              "displayName": "test02"
            },
            {
              "identifier": "identifier_01",
              "displayName": "identifier_01"
            }
          ],
          "events": [
            {
              "outputData": [
                {
                  "identifier": "test01",
                  "index": 0
                }
              ],
              "identifier": "event1",
              "displayName": "abc"
            }
          ]
        }
        ```

        参数说明如下：

        |参数|类型|说明|
        |--|--|--|
        |services|Object|服务|
        |properties|Object|属性|
        |events|Object|事件|
        |outputData|Object|输出参数，如事件上报数据、服务调用的返回结果|
        |identifier|String|描述信息，用于标识。|
        |inputData|Object|输入参数，如服务的入参|
        |index|Integer|索引信息|
        |displayName|String|展示名称|

-   serverList说明

    通道的信息也分为Modbus和OPC UA协议两种。

    -   Modbus协议

        ```
        [
          {
            "baudRate": 1200,
            "protocol": "RTU",
            "byteSize": 8,
            "stopBits": 2,
            "parity": 1,
            "name": "modbus01",
            "serialPort": "0",
            "serverId": "D73251B4277742"
          },
          {
            "protocol": "TCP",
            "port": 8000,
            "ip": "192.168.0.1",
            "name": "modbus02",
            "serverId": "586CB066D6A34"
          }
        ]
        ```

        |参数|类型|说明|
        |--|--|--|
        |protocol|String|协议类型，TCP或者RTU|
        |port|Integer|端口号|
        |ip|String|IP地址|
        |name|String|通道名称|
        |serverId|String|通道的ID|
        |baudRate|Integer|波特率|
        |byteSize|Integer|字节数|
        |stopBits|Integer|停止位|
        |parity|Integer|奇偶校验位。        -   E：偶校验
        -   O：奇校验
        -   N：无校验
 |
        |serialPort|String|串口号|

    -   OPC UA协议

        ```
        {
          "password": "XIJTginONohPEUAyZxLB7Q==",
          "secPolicy": "Basic128Rsa15",
          "name": "server_01",
          "secMode": "Sign",
          "userName": "123",
          "serverId": "55A9D276A7ED470",
          "url": "tcp:00",
          "timeout": 10
        }
        ```

        参数说明如下：

        |参数|类型|说明|
        |--|--|--|
        |password|String|密码，采用AES算法加密。具体说明见下文。|
        |secPolicy|String|加密策略，取值：None，Basic128Rsa15，Basic256|
        |secMode|String|加密模式，取值：None，Sign，SignAndEncrypt|
        |name|String|server名称|
        |userName|String|用户名|
        |serverId|String|server的ID|
        |url|String|服务器连接地址|
        |timeout|Integer|超时时间|

        OPC UA的password加密方式

        加密算法采用AES算法，使用128位（16字节）分组，缺省的mode为CBC，缺省的padding为PKCS5Padding， 秘钥使用设备的deviceSecret，加密后的内容采用BASE64进行编码。

        加解密示例代码：

        ```
         private static String instance = "AES/CBC/PKCS5Padding";
        
            private static String algorithm = "AES";
        
            private static String charsetName = "utf-8";
            /**
             * 加密算法
             *
             * @param data             待加密内容
             * @param deviceSecret     设备的秘钥
             * @return
             */
            public static String aesEncrypt(String data, String deviceSecret) {
                try {
                    Cipher cipher = Cipher.getInstance(instance);
                    byte[] raw = deviceSecret.getBytes();
                    SecretKeySpec key = new SecretKeySpec(raw, algorithm);
                    IvParameterSpec ivParameter = new IvParameterSpec(deviceSecret.substring(0, 16).getBytes());
                    cipher.init(Cipher.ENCRYPT_MODE, key, ivParameter);
                    byte[] encrypted = cipher.doFinal(data.getBytes(charsetName));
        
                    return new BASE64Encoder().encode(encrypted);
                } catch (Exception e) {
                    e.printStackTrace();
                }
        
                return null;
            }
        
            public static String aesDecrypt(String data, String deviceSecret) {
                try {
                    byte[] raw = deviceSecret.getBytes(charsetName);
                    byte[] encrypted1 = new BASE64Decoder().decodeBuffer(data);
                    SecretKeySpec key = new SecretKeySpec(raw, algorithm);
                    Cipher cipher = Cipher.getInstance(instance);
                    IvParameterSpec ivParameter = new IvParameterSpec(deviceSecret.substring(0, 16).getBytes());
                    cipher.init(Cipher.DECRYPT_MODE, key, ivParameter);
                    byte[] originalBytes = cipher.doFinal(encrypted1);
                    String originalString = new String(originalBytes, charsetName);
                    return originalString;
                } catch (Exception ex) {
                    ex.printStackTrace();
                }
        
                return null;
            }
        
            public static void main(String[] args) throws Exception {
                String text = "test123";
                String secret = "testTNmjyWHQzniA8wEkTNmjyWHQtest";
                String data = null;
                data = aesEncrypt(text, secret);
                System.out.println(data);
                System.out.println(aesDecrypt(data, secret));
            }
        ```

-   deviceList说明
    -   Modbus协议

        ```
        {
          "deviceConfig": {
            "slaveId": 1,
            "serverId": "D73251B4277742D"
          },
          "productKey": "test02",
          "deviceName": "test01"
        }
        ```

        参数说明如下：

        |参数|类型|取值|
        |--|--|--|
        |deviceConfig|Object|设备信息|
        |slaveId|Integer|从站ID|
        |serverId|String|通道ID|
        |productKey|String|产品ID|
        |deviceName|String|设备名称|

    -   OPC UA协议

        ```
        {
          "deviceConfig": {
            "displayNamePath": "123",
            "serverId": "44895C63E3FF4013924CEF31519ABE7B"
          },
          "productKey": "test01",
          "deviceName": "test_02"
        }
        ```

        参数说明如下：

        |参数|类型|说明|
        |--|--|--|
        |deviceConfig|Object|设备连接信息|
        |productKey|String|产品ID|
        |deviceName|String|设备名称|
        |displayNamePath|String|展示名称|
        |serverId|String|关联的通道ID|


## 五、设备禁用、删除 {#section_xwd_p1h_12b .section}

**设备禁用**

下行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/disable
-   REPLY TOPIC：/sys/\{productKey\}/\{deviceName\}/thing/disable\_reply

设备禁用提供设备侧的通道能力，云端使用异步的方式推送该消息， 设备订阅该Topic。具体禁用功能适用于网关类型设备，网关可以使用该功能禁用相应的子设备。

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": {},
  "method": "thing.disable"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|请求参数， 为空即可|
|method|String|请求方法|
|code|Integer|结果信息， 具体参考设备端通用code|

**恢复禁用**

下行

-   TOPIC：/sys/\{productKey\}/\{deviceName\}/thing/enable
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/enable\_reply

设备恢复禁用提供设备侧的通道能力，云端使用异步的方式推送消息，设备订阅该topic。具体恢复禁用功能适用于网关类型设备，网关可以使用该功能恢复被禁用的子设备。

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": {},
  "method": "thing.enable"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|请求参数， 为空即可|
|method|String|请求方法|
|code|Integer|结果信息， 具体参考设备端通用code|

**删除设备**

下行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/delete
-   REPLY TOPIC : /sys/\{productKey\}/\{deviceName\}/thing/delete\_reply

删除设备提供设备侧的通道能力，云端使用异步的方式推送消息，设备订阅该topic。具体删除设备的功能适用于网关类型设备，网关可以使用该功能删除相应的子设备。

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": {},
  "method": "thing.delete"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|请求参数， 为空即可|
|method|String|请求方法|
|code|String|结果信息， 具体参考设备端通用code|

## 六、设备标签 {#section_hnn_z1h_12b .section}

**标签信息上报**

上行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/deviceinfo/update
-   REPLY TOPIC： /sys/\{productKey\}/\{deviceName\}/thing/deviceinfo/update\_reply

部分设备信息，如厂商、设备型号等，展示用的静态扩展信息，可以保存为设备标签。

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": [
    {
      "attrKey": "Temperature",
      "attrValue": "36.8"
    }
  ],
  "method": "thing.deviceinfo.update"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明：

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object| 请求参数

 params元素个数不超过200个

 |
|method|String|请求方法|
|attrKey|String| 标签的名称

 -   长度不超过100字节
-   仅允许字符集为\[a-z, A-Z, 0-9\]和下划线
-   首字符必须是字母或者下划线

 |
|attrValue|String|标签的值|
|code|Integer|结果信息， 200表示成功|

错误码

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|6100|device not found|设备不存在|

**删除标签信息**

上行

-   TOPIC /sys/\{productKey\}/\{deviceName\}/thing/deviceinfo/delete
-   REPLY TOPIC /sys/\{productKey\}/\{deviceName\}/thing/deviceinfo/delete\_reply

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": [
    {
      "attrKey": "Temperature"
    }
  ],
  "method": "thing.deviceinfo.delete"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明：

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|请求参数|
|method|String|请求方法|
|attrKey|String| 标签的名称

 -   长度不超过100字节
-   仅允许字符集为\[a-z, A-Z, 0-9\]和下划线
-   首字符必须是字母或者下划线

 |
|attrValue|String|标签的值|
|code|Integer|结果信息， 200表示成功|

错误信息

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|6100|device not found|设备不存在|

## 七、TSL模板 {#section_fr1_fbh_12b .section}

设备可以通过上行请求获取设备的[描述TSL模板](../../../../intl.zh-CN/用户指南/产品与设备/高级版/什么是物模型.md#)。

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/dsltemplate/get
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/dsltemplate/get\_reply

Alink请求数据格式

```
{
  "id": "123",
  "version": "1.0",
  "params": {},
  "method": "thing.dsltemplate.get"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {
    "schema": "https://iot-tsl.oss-cn-shanghai.aliyuncs.com/schema.json",
    "link": "/sys/1234556554/airCondition/thing/",
    "profile": {
      "productKey": "1234556554",
      "deviceName": "airCondition"
    },
    "properties": [
      {
        "identifier": "fan_array_property",
        "name": "风扇数组属性",
        "accessMode": "r",
        "required": true,
        "dataType": {
          "type": "array",
          "specs": {
            "size": "128",
            "item": {
              "type": "int"
            }
          }
        }
      }
    ],
    "events": [
      {
        "identifier": "alarm",
        "name": "alarm",
        "desc": "风扇警报",
        "type": "alert",
        "required": true,
        "outputData": [
          {
            "identifier": "errorCode",
            "name": "错误码",
            "dataType": {
              "type": "text",
              "specs": {
                "length": "255"
              }
            }
          }
        ],
        "method": "thing.event.alarm.post"
      }
    ],
    "services": [
      {
        "identifier": "timeReset",
        "name": "timeReset",
        "desc": "校准时间",
        "inputData": [
          {
            "identifier": "timeZone",
            "name": "时区",
            "dataType": {
              "type": "text",
              "specs": {
                "length": "512"
              }
            }
          }
        ],
        "outputData": [
          {
            "identifier": "curTime",
            "name": "当前的时间",
            "dataType": {
              "type": "date",
              "specs": {}
            }
          }
        ],
        "method": "thing.service.timeReset"
      },
      {
        "identifier": "set",
        "name": "set",
        "required": true,
        "desc": "属性设置",
        "method": "thing.service.property.set",
        "inputData": [
          {
            "identifier": "fan_int_property",
            "name": "风扇整数型属性",
            "accessMode": "rw",
            "required": true,
            "dataType": {
              "type": "int",
              "specs": {
                "min": "0",
                "max": "100",
                "unit": "g/ml",
                "unitName": "毫升"
              }
            }
          }
        ],
        "outputData": []
      },
      {
        "identifier": "get",
        "name": "get",
        "required": true,
        "desc": "属性获取",
        "method": "thing.service.property.get",
        "inputData": [
          "array_property",
          "fan_int_property",
          "batch_enum_attr_id",
          "fan_float_property",
          "fan_double_property",
          "fan_text_property",
          "fan_date_property",
          "batch_boolean_attr_id",
          "fan_struct_property"
        ],
        "outputData": [
          {
            "identifier": "fan_array_property",
            "name": "风扇数组属性",
            "accessMode": "r",
            "required": true,
            "dataType": {
              "type": "array",
              "specs": {
                "size": "128",
                "item": {
                  "type": "int"
                }
              }
            }
          }
        ]
      }
    ]
  }
}
```

参数说明：

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|params|Object|为空即可|
|method|String|请求方法|
|productKey|String|产品的Key,，示例中取值为1234556554|
|deviceName|String|设备名称，示例中取值为airCondition|
|data|Object|物的TSL描述，具体参考[什么是物模型](../../../../intl.zh-CN/用户指南/产品与设备/高级版/什么是物模型.md#)|

错误码

|错误码|消息|描述|
|:--|:-|:-|
|460|request parameter error|请求参数错误|
|6321|tsl: device not exist in product|设备不存在|

## 八、 固件升级 {#section_ivm_fbh_12b .section}

固件升级流程参考[设备OTA开发](../../../../intl.zh-CN//设备OTA开发.md#)和[固件升级](../../../../intl.zh-CN/用户指南/扩展服务/固件升级.md#)。

**固件版本上报**

上行

-   TOPIC: /ota/device/inform/\{productKey\}/\{deviceName\}。设备上报当前使用的固件版本信息。

Alink请求数据格式

```
{
  "id": 1,
  "params": {
    "version": "1.0.1"
  }
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|设备固件的版本信息|

**推送固件信息**

上行

-   TOPIC: /ota/device/upgrade/\{productKey\}/\{deviceName\}

云端通过这个Topic推送固件信息， 设备订阅该Topic可以获得固件信息。

Alink请求数据格式

```
{
  "code": "1000",
  "data": {
    "size": 432945,
    "version": "2.0.0",
    "url": "https://iotx-ota-pre.oss-cn-shanghai.aliyuncs.com/nopoll_0.4.4.tar.gz?Expires=1502955804&OSSAccessKeyId=XXXXXXXXXXXXXXXXXXXX&Signature=XfgJu7P6DWWejstKJgXJEH0qAKU%3D&security-token=CAISuQJ1q6Ft5B2yfSjIpK6MGsyN1Jx5jo6mVnfBglIPTvlvt5D50Tz2IHtIf3NpAusdsv03nWxT7v4flqFyTINVAEvYZJOPKGrGR0DzDbDasumZsJbo4f%2FMQBqEaXPS2MvVfJ%2BzLrf0ceusbFbpjzJ6xaCAGxypQ12iN%2B%2Fr6%2F5gdc9FcQSkL0B8ZrFsKxBltdUROFbIKP%2BpKWSKuGfLC1dysQcO1wEP4K%2BkkMqH8Uic3h%2Boy%2BgJt8H2PpHhd9NhXuV2WMzn2%2FdtJOiTknxR7ARasaBqhelc4zqA%2FPPlWgAKvkXba7aIoo01fV4jN5JXQfAU8KLO8tRjofHWmojNzBJAAPpYSSy3Rvr7m5efQrrybY1lLO6iZy%2BVio2VSZDxshI5Z3McKARWct06MWV9ABA2TTXXOi40BOxuq%2B3JGoABXC54TOlo7%2F1wTLTsCUqzzeIiXVOK8CfNOkfTucMGHkeYeCdFkm%2FkADhXAnrnGf5a4FbmKMQph2cKsr8y8UfWLC6IzvJsClXTnbJBMeuWIqo5zIynS1pm7gf%2F9N3hVc6%2BEeIk0xfl2tycsUpbL2FoaGk6BAF8hWSWYUXsv59d5Uk%3D",
    "md5": "93230c3bde425a9d7984a594ac55ea1e",
    "sign": "93230c3bde425a9d7984a594ac55ea1e",
    "signMethod": "Md5"
  },
  "id": 1507707025,
  "message": "success"
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|message|String|结果信息|
|version|String|设备固件的版本信息|
|size|Long|固件大小，单位字节|
|url|String|固件OSS地址|
|sign|String|固件签名|
|signMethod|String|签名方法， 目前支持Md5，Sha256两种签名方法。|
|md5|String|作为保留字段，兼容老的设备信息。当签名方法为Md5时，除了会给sign赋值外还会给md5赋值。|

**上报升级进度**

上行

-   Topic:/ota/device/progress/\{productKey\}/\{deviceName\}

固件升级过程中设备可以上报固件升级的进度百分比。

Alink请求数据格式

```
{
  "id": 1,
  "params": {
    "step": "-1",
    "desc": "固件升级失败，请求不到固件信息"
  }
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|step|String| 固件升级进度信息

 取值范围：

-   \[1, 100\] 升级进度百分比
-   -1 表示升级失败
-   -2 表示下载失败
-   -3 表示校验失败
-   -4 表示烧写失败

 |
|desc|String|当前步骤的描述信息，如果发生异常可以用此字段承载错误信息|

**设备请求固件信息**

-   TOPIC: /ota/device/request/\{productKey\}/\{deviceName\}

Alink请求数据格式

```
{
  "id": 1,
  "params": {
    "version": "1.0.1"
  }
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|设备固件的版本信息|

## 九、远程配置 {#section_xg4_fbh_12b .section}

远程配置使用请参考[远程配置](https://help.aliyun.com/document_detail/68703.html?spm=a2c4g.11186623.6.697.Uu5IRH)。

**设备主动请求配置信息**

上行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/config/get
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/config/get\_reply

Alink请求数据格式

```
{
  "id": 123,
  "version": "1.0",
  "params": {
    "configScope": "product",
    "getType": "file"
  },
  "method": "thing.config.get"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "version": "1.0",
  "code": 200,
  "data": {
    "configId": "123dagdah",
    "configSize": 1234565,
    "sign": "123214adfadgadg",
    "signMethod": "Sha256",
    "url": "https://iotx-config.oss-cn-shanghai.aliyuncs.com/nopoll_0.4.4.tar.gz?Expires=1502955804&OSSAccessKeyId=XXXXXXXXXXXXXXXXXXXX&Signature=XfgJu7P6DWWejstKJgXJEH0qAKU%3D&security-token=CAISuQJ1q6Ft5B2yfSjIpK6MGsyN1Jx5jo6mVnfBglIPTvlvt5D50Tz2IHtIf3NpAusdsv03nWxT7v4flqFyTINVAEvYZJOPKGrGR0DzDbDasumZsJbo4f%2FMQBqEaXPS2MvVfJ%2BzLrf0ceusbFbpjzJ6xaCAGxypQ12iN%2B%2Fr6%2F5gdc9FcQSkL0B8ZrFsKxBltdUROFbIKP%2BpKWSKuGfLC1dysQcO1wEP4K%2BkkMqH8Uic3h%2Boy%2BgJt8H2PpHhd9NhXuV2WMzn2%2FdtJOiTknxR7ARasaBqhelc4zqA%2FPPlWgAKvkXba7aIoo01fV4jN5JXQfAU8KLO8tRjofHWmojNzBJAAPpYSSy3Rvr7m5efQrrybY1lLO6iZy%2BVio2VSZDxshI5Z3McKARWct06MWV9ABA2TTXXOi40BOxuq%2B3JGoABXC54TOlo7%2F1wTLTsCUqzzeIiXVOK8CfNOkfTucMGHkeYeCdFkm%2FkADhXAnrnGf5a4FbmKMQph2cKsr8y8UfWLC6IzvJsClXTnbJBMeuWIqo5zIynS1pm7gf%2F9N3hVc6%2BEeIk0xfl2tycsUpbL2FoaGk6BAF8hWSWYUXsv59d5Uk%3D",
    "getType": "file"
  }
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|configScope|String|配置范围， 目前只支持产品维度配置。 取值：prodcut|
|getType|String|获取配置类型。 目前支持文件类型，取值：file|
|configId|String|配置的Id|
|configSize|Long|配置大小，按字节计算|
|sign|String|签名|
|signMethod|String|签名方法，仅支持Sha256|
|url|String|配置的oss地址|
|code|Integer|结果码， 200标志成功|

错误码

|错误码|消息|描述|
|:--|:-|:-|
|6713|thing config function is not available|物的配置功能不可用，需要打开物的配置开关|
|6710|no data|没有配置的数据|

**配置推送**

下行

-   TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/config/push
-   REPLY TOPIC: /sys/\{productKey\}/\{deviceName\}/thing/config/push\_reply

设备订阅该Topic后，用户在物联网控制台批量推送配置信息时，云端采用异步推送方式向设备推送信息。设备返回的结果，可以通过数据流转获取， 数据流转topic为`/{productKey}/{deviceName}/thing/downlink/reply/message`。

Alink请求数据格式：

```
{
  "id": "123",
  "version": "1.0",
  "params": {
    "configId": "123dagdah",
    "configSize": 1234565,
    "sign": "123214adfadgadg",
    "signMethod": "Sha256",
    "url": "https://iotx-config.oss-cn-shanghai.aliyuncs.com/nopoll_0.4.4.tar.gz?Expires=1502955804&OSSAccessKeyId=XXXXXXXXXXXXXXXXXXXX&Signature=XfgJu7P6DWWejstKJgXJEH0qAKU%3D&security-token=CAISuQJ1q6Ft5B2yfSjIpK6MGsyN1Jx5jo6mVnfBglIPTvlvt5D50Tz2IHtIf3NpAusdsv03nWxT7v4flqFyTINVAEvYZJOPKGrGR0DzDbDasumZsJbo4f%2FMQBqEaXPS2MvVfJ%2BzLrf0ceusbFbpjzJ6xaCAGxypQ12iN%2B%2Fr6%2F5gdc9FcQSkL0B8ZrFsKxBltdUROFbIKP%2BpKWSKuGfLC1dysQcO1wEP4K%2BkkMqH8Uic3h%2Boy%2BgJt8H2PpHhd9NhXuV2WMzn2%2FdtJOiTknxR7ARasaBqhelc4zqA%2FPPlWgAKvkXba7aIoo01fV4jN5JXQfAU8KLO8tRjofHWmojNzBJAAPpYSSy3Rvr7m5efQrrybY1lLO6iZy%2BVio2VSZDxshI5Z3McKARWct06MWV9ABA2TTXXOi40BOxuq%2B3JGoABXC54TOlo7%2F1wTLTsCUqzzeIiXVOK8CfNOkfTucMGHkeYeCdFkm%2FkADhXAnrnGf5a4FbmKMQph2cKsr8y8UfWLC6IzvJsClXTnbJBMeuWIqo5zIynS1pm7gf%2F9N3hVc6%2BEeIk0xfl2tycsUpbL2FoaGk6BAF8hWSWYUXsv59d5Uk%3D",
    "getType": "file"
  },
  "method": "thing.config.push"
}
```

Alink响应数据格式

```
{
  "id": "123",
  "code": 200,
  "data": {}
}
```

参数说明

|参数|取值|说明|
|:-|:-|:-|
|id|String|消息ID号，保留值|
|version|String|协议版本号，目前协议版本1.0|
|configScope|String|配置范围， 目前只支持产品维度配置。 取值：prodcut|
|getType|String|获取配置类型，目前支持文件类型，取值：file|
|configId|String|配置的Id|
|configSize|Long|配置大小，按字节计算|
|sign|String|签名|
|signMethod|String|签名方法，仅支持Sha256|
|url|String|配置的oss地址|
|code|Integer|结果信息， 具体参考设备端通用code|

## 十、设备端通用code {#section_bqp_fbh_12b .section}

设备通用code信息，用于表达云端下行推送时设备侧业务处理的返回结果。

|错误码|消息|描述|
|:--|:-|:-|
|200|success|请求成功|
|400|request error|内部服务错误， 处理时发生内部错误|
|460|request parameter error|请求参数错误， 设备入参校验失败|
|429|too many requests|请求过于频繁，设备端处理不过来时可以使用|
|100000-110000| |从100000到110000用于设备自定义错误信息，和云端错误信息加以区分|

