# ListRuleActions {#reference_jtm_xjd_xdb .reference}

You can call this operation to retrieve all rule actions of a specified rule.

## Request parameters {#section_e2c_4t1_ydb .section}

|Name|Type|Required|Description|
|:---|:---|:-------|:----------|
|Action|String|Yes.|The operation that you want to perform. Set the value to ListRuleActions.|
|RuleId|String|Yes|The ID of the rule that you want to query.|

## Response parameters {#section_cyf_wt1_ydb .section}

|Name|Type|Description |
|:---|:---|:-----------|
|RequestId|String|The GUID generated by Alibaba Cloud for the request.|
|Success|Boolean|Indicates whether the call is successful. A value of true indicates that the call is successful. A value of false indicates that the call has failed. |
|ErrorMessage|String|The error message returned when the call fails.|
|RuleActionList|RuleActionInfo|The rule action information returned when the call is successful. For more information, see RuleActionInfo.|

|Name|Type|Description |
|:---|:---|:-----------|
|Id|Long|The rule action ID.|
|RuleId|Long|The ID of the rule that includes this rule action.|
|Type|String| The rule action type. Values include:

 REPUBLISH: Forward to another topic.

 OTS: Save to Table Store.

 MNS: Send to Message Service.

 ONS: Send to Message Queue.

 HiTSDB: Save to High-Performance Time Series Database.

 FC: Send to Function Compute.

 DATAHUB: Send to DataHub.

 RDS: Save to cloud databases.

 |
|Configuration|String|The configurations of this rule action.|

## Examples {#section_uz3_j51_ydb .section}

**Request parameters**

```
https://iot.cn-shanghai.aliyuncs.com/?Action=ListRuleActions
&RuleId=10000
&<common request parameters>
```

**Response example**

```
{
    "RequestId": "21D327AF-A7DE-4E59-B5D1-ACAC8C024555",
    "RuleActionList": [{
        "Configuration": "{\"topic\":\"/a1GM***/bike/get\",\"uid\":\"3216***\"}",
        "Id": 10001,
        "RuleId": 1000,
        "Type": REPUBLISH
    },
    {
        "Configuration": "{\"endPoint\":\"http://ons.aliyun.com:8080/rocketmq/nsaddr4client\",\"regionName\":\"cn-shanghai\",\"topic\":\"test_ons\",\"uid\":\"3216***\"}",
        "Id": 10002,
        "RuleId": 1000,
        "Type": ONS
    }],
    "Success": true
}
```

