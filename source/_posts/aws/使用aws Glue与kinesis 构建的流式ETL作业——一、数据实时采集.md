---
title: 使用aws Glue与kinesis 构建的流式ETL作业——一、数据实时采集
tags: 
- AWS
- 大数据
- ETL
categories:
- AWS
date: 2022-09-09 17:42:39
---
## 一、数据采集准备工作
- [一、数据采集准备工作](#一数据采集准备工作)
  - [1.1、研究的背景](#11研究的背景)
  - [1.2、使用Glue构建流式ETL的原因](#12使用glue构建流式etl的原因)
  - [1.3、无服务器流式ETL架构](#13无服务器流式etl架构)
  - [1.4、简易架构](#14简易架构)
  - [1.5、Kinesis Data Stream创建](#15kinesis-data-stream创建)
  - [1.6、CloudWatch数据筛选](#16cloudwatch数据筛选)
  - [1.7、KDS中的数据验证](#17kds中的数据验证)
  - [1.8、总结](#18总结)
- [作者](#作者)


### 1.1、研究的背景

为了可以更高效的从项目的数据集中提取更有意义的数据，并进行统计分析。

### 1.2、使用Glue构建流式ETL的原因

1、项目部署在AWS。
2、Amazon Glue中的流式ETL基于Apache Spark的结构化流引擎，该引擎提供一种高容错、可扩展且易于实现的方法，能够实现端到端的流处理。

### 1.3、无服务器流式ETL架构
在此流式ETL架构中，将使用lambda模拟日志创建cloudwatch指标并将其以流的形式发布至Kinesis Data Streams当中。我们还将在Amazon Glue中创建一项流式ETL作业，该作业以微批次的形式获取连续生成的stream数据，对数据进行转换、聚合，并将结果传递至接收器，最终利用这部分结果显示可视化图表或在下游流程中继续使用。

### 1.4、简易架构
![](https://img-blog.csdnimg.cn/8ac3b792ec034effb144e10c66782078.png)

### 1.5、Kinesis Data Stream创建
Kinesis Data Stream的用处？：实时捕获数据
从数十万个数据源提取并存储数据流:
&nbsp;&nbsp;&nbsp;&nbsp;日志和事件数据采集
&nbsp;&nbsp;&nbsp;&nbsp;IoT 设备数据捕获
&nbsp;&nbsp;&nbsp;&nbsp;移动数据采集
&nbsp;&nbsp;&nbsp;&nbsp;游戏数据源
<font color="#ff0000">此案例中，我们将从CloudWatch中进行数据采集
| 步骤                                            | 图例                                                                  |
| ----------------------------------------------- | --------------------------------------------------------------------- |
| **1、入口**                                     | ![](https://img-blog.csdnimg.cn/674eba6985bf4b939c901144d071a0ec.png) |
| **2、创建（按需模式无需手动预置和扩展数据流）** | ![](https://img-blog.csdnimg.cn/f7751d9bba43428e93593e785fbdc276.png) |


### 1.6、CloudWatch数据筛选
<font color="#ff00ff">前置条件：已准备好用来进行数据采集的CloudWatch</font>
我们将会在某个CloudWatch日志组中创建<font color="#ff00">日志筛选条件
| 步骤                                                                                                                        | 图例                                                                                               |
| --------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| **1、入口**                                                                                                                 | ![](https://img-blog.csdnimg.cn/6ad849aae264404e9a1e16364fe4ddb6.png)                              |
| **2、选择上步中创建的KDS**                                                                                                  | ![](https://img-blog.csdnimg.cn/666c7bfb6f584db0b7dd4c1358222870.png)                              |
| **3、IAM角色（需要有Kinesis Data Stream的权限）**                                                                           | ![](https://img-blog.csdnimg.cn/5f0628e9dbd145b78b5062e9e80cb98b.png)权限与实体见下方“IAM角色权限” |
| **4、配置筛选条件（可根据日志格式自定义）**（<font color="#ff00ff">例如：图中配置为筛选包含"is_save_kinesis"的数据</font>） | ![](https://img-blog.csdnimg.cn/9a084f4a139647cb905e0d8f50d2b352.png)                              |
| **5、测试数据（可以选定某条日志流，或自定义数据进行测试结果显示）**                                                         | ![](https://img-blog.csdnimg.cn/552cfc470dfb4449b9da71b2128d1b66.png)                              |
| **6、完成日志筛选条件创建（<font color="#f0f">每个日志组最多只能创建两条</font>）**                                         | ![](https://img-blog.csdnimg.cn/a84033ce693a47d48651a69a8d0cc2b0.png)                              |

IAM角色权限：
&nbsp;&nbsp;&nbsp;&nbsp;可信实体：
```xml
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "logs.【区域】.amazonaws.com"
            },
            "Action": "sts:AssumeRole",
            "Condition": {
                "StringLike": {
                    "aws:SourceArn": "【CloudWatch的ARN】"
                }
            }
        }
     ]
}
```

&nbsp;&nbsp;&nbsp;&nbsp;策略：

```xml
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "WriteOutputKinesis",
            "Effect": "Allow",
            "Action": [
                "kinesis:DescribeStream",
                "kinesis:PutRecord",
                "kinesis:PutRecords"
            ],
            "Resource": [
                "【Kinesis Data Stream的ARN】"
                
            ]
        }
    ]
}
```


### 1.7、KDS中的数据验证
<font color="#f0f">前置条件：一个已绑定上 以Kinesis作为触发器的Lambda实例</font>
<font color="#f00">此案例也可使用Lambda来实现数据流的处理。每当Kinesis Data Stream中传入数据时，就会触发绑定了KDS的Lambda，有Lambda来清洗、转换、存储数据。</font>
在我们向该CloudWatch中发送一条日志数据后，将会在Kinesis Data Stream控制台监控到数据的流入
![](https://img-blog.csdnimg.cn/14eb4f74c1ff45a9b7e6bb204714cc90.png)
![](https://img-blog.csdnimg.cn/b4e140bf32334b74846ee3a72e480e62.png)
接下来，我们将会验证解析一下Kinesis Data Stream中的数据与格式。
原始数据存储在event.Records[0].kinesis.data中（下一步的ETL工作中，我们会从此处获取数据）
验证代码：

```python
def lambda_handler(event, context):

    raw_kinesis_records = event['Records']

    # records = deaggregate_records(raw_kinesis_records)
    records = raw_kinesis_records
    i=0
    for record in records:
        print("this is record")
        #Kinesis data is base64 encoded so decode here
        payload=base64.b64decode(record["kinesis"]["data"],validate=False)
        
        data = gzip.decompress(payload).decode("utf-8")
		print(data)
        i+=1
        
    print(i)
```
**结果：**
<font color="#f0f">&nbsp;&nbsp;&nbsp;&nbsp;其中的message为我们的原始数据的<font color="#f00">字符串</font></font>
![](https://img-blog.csdnimg.cn/806dc97c2cda4b66b501224cd680086a.png)

### 1.8、总结

在此案例中，我们使用了CloudWatch + Kinesis Data Stream完成了前期的数据实时采集的工作，并且，使用了Lambda来作为触发器来对数据进行了一个验证操作（也可使用Lambda来进行ETL工作）。

## 作者

Zikuo Su
