---
title: 基于Glue ETL(提取、转换和加载)的serviceless 数据分析——三、serverless数据分析
tags: 
- AWS
- 大数据
- ETL
categories:
- AWS
---

# 三、serverless数据分析


- [三、serverless数据分析](#三serverless数据分析)
  - [1、创建Lambda](#1创建lambda)
  - [2、创建API Gateway](#2创建api-gateway)
  - [3、结果](#3结果)
  - [总结](#总结)

## 1、创建Lambda
<font color=##00aa00>在Lambda中，我们将使用python3作为代码语言。
| 步骤                                                                                | 图例                                                                  |
| ----------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| 1、入口                                                                             | ![](https://img-blog.csdnimg.cn/b634fb87a6444d5b92edf7b4ccbcdb0a.png) |
|                                                                                     |
| 2、创建（我们选择使用python3.7）                                                    | ![](https://img-blog.csdnimg.cn/96278d761c66480ca1172171b0daed86.png) |
|                                                                                     |
| 3、IAM权限（权限可信实体需要包括lambda才能将角色绑定到Lambda上）                    | ![](https://img-blog.csdnimg.cn/55ae86c52d9b40e2946c9a6fd9093a7e.png) |
|                                                                                     |
| IAM可信实体                                                                         | 见下方                                                                |
|                                                                                     |
| 4、指定处理函数（处理程序要为用户程序的入口）                                       | ![](https://img-blog.csdnimg.cn/7b49a0a6f409413ab95ba8a39a24216f.png) |
|                                                                                     |
| 5、添加层（层为我们的代码运行时的环境，并且，兼容运行时要包含上一步中的运行时环境） | ![](https://img-blog.csdnimg.cn/4183f4a174db47f8a3bf9a691a6778dd.png) |
|                                                                                     |
| 6、代码（在此代码中使用了boto3来连接Athena，可自定义sql，使用方法请看官方文档）     |
| 见下方                                                                              |

**可信实体：**

```python
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "lambda.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

Lambda代码

```python
import boto3, os, json
import pandas as pd

from pyathena import connect
import time
REGION = "us-west-2"

# expected request: anomaly/{meter_id}?data_start={}&data_end={}&outlier_only={}
def lambda_handler(event, context):
    ATHENA_OUTPUT_BUCKET = "xxx/athena"
    DB_SCHEMA = "suzikuo_test_db"

    USE_WEATHER_DATA = 0
    pathParameter = event["pathParameters"]
    queryParameter = event["queryStringParameters"]
    METER_ID = pathParameter['meter_id']
    DATA_START = queryParameter['data_start']
    DATA_END = queryParameter['data_end']
    OUTLIER_ONLY = queryParameter['outlier_only']
    query = '''
    select * from "{}".reading_type_int
    where meter_id = '{}'
    and cast(reading_date_time as timestamp) >= timestamp '{}' and cast(reading_date_time as timestamp) < timestamp '{}'
    '''.format(DB_SCHEMA, METER_ID, DATA_START, DATA_END)

    athena = boto3.client('athena')
    response = athena.start_query_execution(
    QueryString=query,
    QueryExecutionContext={
        'Database': 'suzikuo_test_db'
    },
    ResultConfiguration={
        'OutputLocation': 's3://suzikuo-test-2022-8-4-s3/athena',
        'EncryptionConfiguration': {
            'EncryptionOption': 'SSE_S3'
        }
    }
    )
    while True:
        try:
            query_results = athena.get_query_results(
                QueryExecutionId=response['QueryExecutionId']
            )
            break
        except Exception as err:
            if 'Query has not yet finished' in str(err):
                time.sleep(3)
            else:
                raise(err)

    return query_results['ResultSet']['Rows']
```

<br><br><br><br><br>

## 2、创建API Gateway
<font color=##00aa00>API Gateway +Lambda 可轻松实现一个serverless架构

| 步骤                                          | 图例                                                                  |
| --------------------------------------------- | --------------------------------------------------------------------- |
| 1、入口                                       | ![](https://img-blog.csdnimg.cn/cd56889829294111a351648a90236c8d.png) |
|                                               |
| 2、API（我们使用的是Lambda，所以选HTTP API）  | ![](https://img-blog.csdnimg.cn/500335f48be04de9b98ac1ea0ce31922.png) |
|                                               |
| 3、创建集成（指定要绑定的Lambda）             | ![](https://img-blog.csdnimg.cn/f51dd20ca63f45828ba271a5230e7508.png) |
|                                               |
| 4、配置路由（指定路由要请求的集成（lambda）） | ![](https://img-blog.csdnimg.cn/3badbf6467354357822dbeda2df43414.png) |
|                                               |
| 5、一直下一步即可                             |                                                                       |

<br>
<br><br>

## 3、结果
<font color=##aa0000>此案例只查询了某一ID的某个时间段内的数据</font>
<font color=##00aa00>
通过获取URI和参数，在Lambda中编写逻辑，可以实现我们对数据的任意操作。
</font>

![](https://img-blog.csdnimg.cn/93bc2063514a4cd1b34b503be6c1ad3f.png)

## 总结
**到此，我们已经完成了基于Glue ETL(提取、转换和加载)的serviceless 数据分析的全部过程了。在此案例中，我们使用到了AWS 服务中的Glue、S3、APIGateway、Lambda等服务实现了一个通过API访问的数据统计与分析接口。**
