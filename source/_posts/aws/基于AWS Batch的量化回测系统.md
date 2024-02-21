---
title: 基于AWS Batch的量化回测系统
tags: 
- AWS
- 量化回测

categories:
- AWS
---

# 1 Amazon SageMaker
- [1 Amazon SageMaker](#1-amazon-sagemaker)
  - [1.1 创建笔记本实例](#11-创建笔记本实例)
  - [1.2 上传ipynb](#12-上传ipynb)
  - [1.3 进入实例](#13-进入实例)
  - [1.4 选择JupyterLab解释器语言](#14-选择jupyterlab解释器语言)
  - [1.5 修改IAM角色权限](#15-修改iam角色权限)
  - [1.6 执行代码](#16-执行代码)
- [2 AWS Batch](#2-aws-batch)
  - [2.1 创建计算环境](#21-创建计算环境)
  - [2.2 创建任务队列](#22-创建任务队列)
  - [2.3 创建任务定义](#23-创建任务定义)
  - [2.4 创建执行任务](#24-创建执行任务)
  - [2.5 执行结果](#25-执行结果)
- [参考文献](#参考文献)
- [作者](#作者)

## 1.1 创建笔记本实例
[Amazon SageMaker Notebook](https://ap-northeast-1.console.aws.amazon.com/sagemaker/home?region=ap-northeast-1#/notebook-instances) 实例是一个用来进行机器学习 (ML) 计算实例，用来运行Jupyter Notebook 应用程序。我们可以在其网页中交互式的编写代码和运行代码，并且可以直接返回逐段代码的运行结果。
接下来，我们将会创建一个笔记本实例。
输入笔记本实例名称。
![](https://img-blog.csdnimg.cn/e668dc2773dc4d93b4909c2a5421c138.png)
对于[IAM角色](https://us-east-1.console.aws.amazon.com/iamv2/home#/roles)，我们可以选择创建新角色
![](https://img-blog.csdnimg.cn/ce890652a08c44758369e7053ba196cb.png)
最后，点击创建笔记本实例。
## 1.2 上传ipynb
[demo文件git地址](https://gitee.com/where-is-my-diao-chan/public_document_collection/blob/master/aws_files/backtest%20(1).ipynb)
打开 [Jupyter](https://ap-northeast-1.console.aws.amazon.com/sagemaker/home?region=ap-northeast-1#/notebook-instances)
![](https://img-blog.csdnimg.cn/1baaf54010ad434fb289c99164373307.png)
上传ipynb文件
![](https://img-blog.csdnimg.cn/6d8dd7ba597d41bc863454fe607248ba.png)
## 1.3 进入实例
![](https://img-blog.csdnimg.cn/1b58464e5d964a44a7f31bf8559eff71.png)

## 1.4 选择JupyterLab解释器语言
选择使用"conda_python3"。
![](https://img-blog.csdnimg.cn/0bb13f1c8a914a39bed6b191866cd04a.png)
## 1.5 修改IAM角色权限
eg: 为了方便测试，暂时先赋予全部权限。（生产环境中要严格遵从最小权限原则。）

找到我们“1.1.1 创建笔记本实例”中创建的[IAM角色](https://us-east-1.console.aws.amazon.com/iamv2/home#/roles)，为此角色赋予以下权限。
```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "ecr:GetRegistryPolicy",
                "ecr:CreateRepository",
                "ecr:DescribeRegistry",
                "ecr:DescribePullThroughCacheRules",
                "ecr:GetAuthorizationToken",
                "ecr:PutRegistryScanningConfiguration",
                "ecr:CreatePullThroughCacheRule",
                "ecr:DeletePullThroughCacheRule",
                "ecr:PutRegistryPolicy",
                "ecr:GetRegistryScanningConfiguration",
                "ecr:BatchImportUpstreamImage",
                "ecr:DeleteRegistryPolicy",
                "ecr:PutReplicationConfiguration"
            ],
            "Resource": "*"
        },
        {
            "Sid": "VisualEditor1",
            "Effect": "Allow",
            "Action": "ecr:*",
            "Resource": "arn:aws:ecr:【aws_region】:【aws_account_id】:repository/【repository_name】"
        }
    ]
}
```
## 1.6 执行代码
使用“Shift+Enter”组合键来执行当前选择的代码块。执行结果将会在代码块下方展示。
![](https://img-blog.csdnimg.cn/a41e4564bfa149ccb9c90399dd532f1b.png)

当我们执行完“将容器推送到远程的ECR镜像仓库”的代码后，我们将会在[Amazon ECR](https://us-west-2.console.aws.amazon.com/ecr/repositories?region=us-west-2)中看到我们创建的镜像仓库。
![](https://img-blog.csdnimg.cn/7502bdedfa724adaba72438ffc0a54f4.png)
![](https://img-blog.csdnimg.cn/df33f2d3ba0d4eb985491abf4678edc9.png)

# 2 AWS Batch
Batch 可以帮助我们在 AWS 云上运行批量计算工作负载。
## 2.1 创建计算环境
前往[Batch](https://ap-northeast-1.console.aws.amazon.com/batch/home?region=ap-northeast-1#compute-environments) 创建计算环境。
![](https://img-blog.csdnimg.cn/0246adbab07a4c3db874aaa3ba32b572.png)

输入计算环境名称
![](https://img-blog.csdnimg.cn/7a7e3fbf660c4ae2812f24040b92aa37.png)
实例配置请根据实际情况进行配置。
![](https://img-blog.csdnimg.cn/e8acb75dbd2c4058853ca955c2f671d5.png)
联网默认即可
![](https://img-blog.csdnimg.cn/57b3b1761d984fe5a94c426c45b0d978.png)
完成创建。

## 2.2 创建任务队列
![](https://img-blog.csdnimg.cn/33c1a3987f31466c9de7c59f05edc80f.png)
输入作业队列名称
![](https://img-blog.csdnimg.cn/7c4b5afe8eb744eb887bb19d0496d6d7.png)
连接的计算环境，请选择上一步创建的计算环境
![](https://img-blog.csdnimg.cn/c74491028725456bbcfc88ae8b3abeec.png)
完成创建。
## 2.3 创建任务定义
![](https://img-blog.csdnimg.cn/dce04efeeb3a47aba76f5908a5a668b2.png)
任务类型我们选择单节点
![](https://img-blog.csdnimg.cn/c41e88e1afea4157b0ada0a8d6072df1.png)
输入名称与超时时间
![](https://img-blog.csdnimg.cn/1c42d870d24c45d58aaee1470d93cd78.png)
选择使用Fargate作为运行环境，并开启分配公共IP。
![](https://img-blog.csdnimg.cn/ae1494cbd29f4735968be855a0faf9a0.png)
进入[ECR](https://ap-northeast-1.console.aws.amazon.com/ecr/repositories?region=ap-northeast-1)，选择我们刚才创建的镜像仓库，复制其ARN。
![](https://img-blog.csdnimg.cn/ab743662582c4f5dacdc87ad843011c0.png)
返回创建，输入映像与命令
命令：python backtest.py 【数据源所在S3桶名】 【要回测的源数据文件名】 【结果存储S3桶名】
![](https://img-blog.csdnimg.cn/e6a6d5a60cec449dae37dc47046d4468.png)
输入重试次数
![](https://img-blog.csdnimg.cn/265042dc8cc7469ba140cd7270203597.png)
完成创建。
## 2.4 创建执行任务
![](https://img-blog.csdnimg.cn/9eabb2adfb464b538fe3a942ddebac12.png)
输入作业名称与运行的作业队列，其他保持默认即可。
![](https://img-blog.csdnimg.cn/81e2a927cfb444f79568fbfe7e4940e1.png)
提交任务。等待作业执行。

## 2.5 执行结果
![](https://img-blog.csdnimg.cn/9038f7c9ab1f4f5da831284a2de556a3.png)
![](https://img-blog.csdnimg.cn/976b5228610c4788916fa2bcec11ea47.png)
# 参考文献
[https://aws.amazon.com/cn/blogs/china/building-a-quantitative-backtesting-system-based-on-aws-batch/?nc1=h_ls](https://aws.amazon.com/cn/blogs/china/building-a-quantitative-backtesting-system-based-on-aws-batch/?nc1=h_ls)
# 作者
ZiKuo Su
