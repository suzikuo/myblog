---
title: 基于Glue ETL(提取、转换和加载)的serviceless 数据分析——一、Glue创建.
tags: 
- AWS
- 大数据
- ETL
categories:
- AWS
---
# 一、通过Athena查询s3中的数据
**此实验使用s3作为数据源**

> ETL: 
> 
>  E   &nbsp;&nbsp; extract  &nbsp;&nbsp;   &nbsp;&nbsp;&nbsp;&nbsp; 输入
>  T   &nbsp;&nbsp; transform &nbsp;&nbsp;&nbsp; 转换
>  L   &nbsp;&nbsp; load         &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  输出

<br>

- [一、通过Athena查询s3中的数据](#一通过athena查询s3中的数据)
  - [简易架构图](#简易架构图)
  - [1、创建Glue数据库](#1创建glue数据库)
  - [2、创建爬网程序](#2创建爬网程序)
  - [3、创建表](#3创建表)
    - [1、爬网程序创建表](#1爬网程序创建表)
    - [2、手动创建表](#2手动创建表)
  - [5、Athena查询](#5athena查询)
  - [6、总结](#6总结)


## 简易架构图
![](https://img-blog.csdnimg.cn/00cecfb1050a4e9087ec188dfdef74f5.png)




## 1、创建Glue数据库
**首先我们需要创建一个数据库**
**我们将会使用爬网程序来填充我们的数据目录**

| 步骤                                              | 图例                                                                  |
| ------------------------------------------------- | --------------------------------------------------------------------- |
| **1、入口**                                       | ![](https://img-blog.csdnimg.cn/a9e3526803ae4a8696b5a1a193dd98fe.png) |
| **2、创建数据库**  **只需输入一个数据库名称即可** | ![](https://img-blog.csdnimg.cn/ecbe4e887d8a45a6ba4ce872d918ce14.png) |
|                                                   |
| 3、创建                                           | ![](https://img-blog.csdnimg.cn/fe6f3ce584af4e8f84aad4c627cc0a17.png) |
|                                                   |

<br>
<br>
<br>
<br>
<br>
<br>

## 2、创建爬网程序
**在任务中，我们经常会使用Glue爬网程序来填充我们的数据目录。
爬虫可以在一次运行中爬取多个数据存储。在爬取完成后，我们会在数据目录中看到由爬虫创建的一个或多个表。
创建表后，我们就可以在接下来的Athena查询或ETL作业中使用表来作为源或目标了。**

| 步骤                                                                                              | 图例                                                                  |
| ------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **1、入口**                                                                                       | ![](https://img-blog.csdnimg.cn/751d75f7050a4f8094d8829d1a1822b6.png) |
| **2、爬虫名称**                                                                                   | ![](https://img-blog.csdnimg.cn/c0b70bee915d4a76aaaac544472efb06.png) |
| **3、选择数据源类型  选择爬取类型（全部爬取，只爬取更新，特定事件）**                             | ![](https://img-blog.csdnimg.cn/a1ffee7b63e74b03835adabf41bfd547.png) |
| **4、选择s3  （可对s3中的需要爬取的数据进行筛选）**                                               | ![](https://img-blog.csdnimg.cn/a431767944b5435396d2fa11ccbba2d1.png) |
| **5、创建或选择爬网程序IAM角色（需要有对应S3与Glue的权限）**                                      | ![](https://img-blog.csdnimg.cn/40b749299f424b15b98e0cf6b3227587.png) |
| **6、对于不确定的实时数据或许要定时更新的数据，可按需选择频率；若只需创建表结构，可选择按需运行** | ![](https://img-blog.csdnimg.cn/bcbb1c24cac04e78961d47b080b6cd9e.png) |
| **7、确认**                                                                                       | ![](https://img-blog.csdnimg.cn/3bf9d288e66a4c7688f584dab361d9bc.png) |
|                                                                                                   |

**此时，数据库与爬网程序已准备完毕**
**我们将会运行爬网程序自动分析数据结构并创建表**
<br>
<br>
<br>
## 3、创建表
	

### 1、爬网程序创建表

| 步骤                                                                                                                                                           | 图例                                                                  |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **1、运行**   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | ![](https://img-blog.csdnimg.cn/a2c9f0f94b9a4f3688c003ee31c407ba.png) |
|                                                                                                                                                                |
| **2、运行中**                                                                                                                                                  | ![](https://img-blog.csdnimg.cn/bf045216015549758ad81c891dc76fb9.png) |
|                                                                                                                                                                |
| **3、运行完毕**                                                                                                                                                | ![](https://img-blog.csdnimg.cn/aed299dfdeac4f158c856288a2779d71.png) |
|                                                                                                                                                                |
| **4、运行结果**                                                                                                                                                | ![](https://img-blog.csdnimg.cn/cce8306bb518477fb02fbc2d68ffe15f.png) |
|                                                                                                                                                                |
| **5、表结构**                                                                                                                                                  | ![](https://img-blog.csdnimg.cn/5121575df8c54bc797f7c1a7637f0260.png) |
|                                                                                                                                                                |

### 2、手动创建表
| 步骤                                          | 图例                                                                  |
| --------------------------------------------- | --------------------------------------------------------------------- |
| **1、入口**                                   | ![](https://img-blog.csdnimg.cn/0b576974392c4e71bae84e162ec03f3f.png) |
|                                               |
| **2、表名**                                   | ![](https://img-blog.csdnimg.cn/34f5fc5883204b53b386209e3dbd8191.png) |
|                                               |
| **3、数据源**                                 | ![](https://img-blog.csdnimg.cn/650121ecaeca47828c85883090ec95d0.png) |
|                                               |
| **4、选择文件类型**                           | ![](https://img-blog.csdnimg.cn/5f19554535684ee99842a8be4d002ee6.png) |
|                                               |
| **5、手动创建表需要自定义列；请根据提示创建** | ![](https://img-blog.csdnimg.cn/a3ed856de2af4fd9a66c48038d20c21c.png) |
|                                               |
| **6、一直下一步即可**                         |                                                                       |

<br>
<br>
<br>
<br>
<br>
<br>

## 5、Athena查询
Athena是一种交互式查询服务（不是数据库）。并且Athena可以使用标准SQL直接查询s3中的数据，前提是需要使用Glue连接s3源。Athena还支持查询如DynamoDB、Redshift、MySQL等数据库。
| 步骤                                    | 图例                                                                  |
| --------------------------------------- | --------------------------------------------------------------------- |
| **1、入口**                             | ![](https://img-blog.csdnimg.cn/19d39af6cf864cec95c225afab042507.png) |
|                                         |
| **2、设置查询结果存储位置：s3**         | ![](https://img-blog.csdnimg.cn/11cf7c82250c4e8e9a766b0a677794a5.png) |
|                                         |
|                                         | ![](https://img-blog.csdnimg.cn/a1e75f6a48ef4c3291a9fd05ff7baba5.png) |
|                                         |
| **3、查看表，可查看数据库以及其中的表** | ![](https://img-blog.csdnimg.cn/557c5b997eab40f19e655b6e4f08c9c1.png) |
|                                         |
| **4、查询结果：使用sql查询**            | ![](https://img-blog.csdnimg.cn/d00288962db3488fa56e94182b43e60f.png) |
|                                         |

## 6、总结
**在此实验中，我们使用Glue 的爬网程序自动解析存储在s3桶中的原始数据，自动创建了表。通过Glue数据库中的表，我们可以使用Athena对表进行查询（Athena每次检索表对应的s3桶数据，按检索量收费）。接下来我们会对原始数据进行转换、清洗以及分区操作，以及使用API Gateway+lambda实现一个无服务架构，通过API查询数据。**
