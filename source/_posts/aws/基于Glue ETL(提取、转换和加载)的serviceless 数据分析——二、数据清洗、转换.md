---
title: 基于Glue ETL(提取、转换和加载)的serviceless 数据分析——二、数据清洗、转换
tags: 
- AWS
- 大数据
- ETL
categories:
- AWS
---

# 二、数据清洗、转换

**此实验使用s3作为数据源**

> ETL: 
> 
>  E   &nbsp;&nbsp; extract  &nbsp;&nbsp;   &nbsp;&nbsp;&nbsp;&nbsp; 输入
>  T   &nbsp;&nbsp; transform &nbsp;&nbsp;&nbsp; 转换
>  L   &nbsp;&nbsp; load         &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;  输出

- [二、数据清洗、转换](#二数据清洗转换)
  - [简易架构图](#简易架构图)
  - [1、数据清洗](#1数据清洗)
  - [2、数据分区](#2数据分区)
  - [3、总结](#3总结)


## 简易架构图
![](https://img-blog.csdnimg.cn/72acdbce3c594d3fac88bcd9aa553b58.png)



## 1、数据清洗
<font color=##00aa00>此步会将s3中的原始数据清洗成我们想要的自定义结构的数据。之后，我们可通过<font color="#ff0000">APIGateway+Lambda+Athena</font>来实现一个无服务器的数据分析服务。
| 步骤                                                                                                                                    | 图例                                                                                                                                                                                                                                                                              |
| --------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1、入口                                                                                                                                 | ![](https://img-blog.csdnimg.cn/87d10ca8a11c4365af2ff9cca526eb1e.png)                                                                                                                                                                                                             |
|                                                                                                                                         |
| 2、创建Job（s3作为数据源，则Type选择Spark，若为Kinesis等，选择Stream Spark）                                                            | ![](https://img-blog.csdnimg.cn/592eb7721a81429697b8f9464c6b9932.png)                                                                                                                                                                                                             |
| 3、IAM角色需要有s3与Glue的权限                                                                                                          | ![](https://img-blog.csdnimg.cn/5d1800dab03542d3a8be59f093572360.png)                                                                                                                                                                                                             |
| 4、选择s3脚本位置,若已经完成脚本的编写工作，则可以选择第二项或第三项，若无则Glue会提供默认脚本                                          | ![](https://img-blog.csdnimg.cn/7d812c8f9ef9488484de0b1cf174477c.png)                                                                                                                                                                                                             |
|                                                                                                                                         |
| 5、安全配置参数                                                                                                                         | ![](https://img-blog.csdnimg.cn/ff3ac1ddd2764207ad665a7a2c74405a.png)**建议：添加参数--enable-auto-scaling为true。每次在我们执行Job任务时，会根据运行 ETL 任务的数据处理单元（DPU）的个数来分配动态IP，在我们子网的动态IP数低于DPU数时，Job将会执行失败。此参数将会动态分配IP。** |
|                                                                                                                                         |
| 6、数据源（）                                                                                                                           | ![](https://img-blog.csdnimg.cn/96ef1ffee060463a920a7ed2e8889d56.png)                                                                                                                                                                                                             |
|                                                                                                                                         |
| 7、数据目标（我们会将清洗后的数据存储到新的s3桶）                                                                                       | ![](https://img-blog.csdnimg.cn/14484dc005774b07afffa19a774715d3.png)                                                                                                                                                                                                             |
|                                                                                                                                         |
| 8、设计架构（在本案例中，我们会自定义脚本。所以不再在此处设计架构）<font color="#ff0000">（此处设计后，脚本会自动生成相关代码）</font > | ![](https://img-blog.csdnimg.cn/df3c0aff2534416f91f29402fb443d10.png)                                                                                                                                                                                                             |
|                                                                                                                                         |
| 9、保存                                                                                                                                 | ![](https://img-blog.csdnimg.cn/cdcd23fdefe9490e8e6a3ec37a9f2eab.png)                                                                                                                                                                                                             |
|                                                                                                                                         |

**编辑脚本**
<font color=##FFFF00>  脚本中的args参数的键值需要从Job的安全配置参数中定义  </font> 


<font color=##00aa00> 1. 连接数据源（s3）
 

```python
#数据源
datasource = glueContext.create_dynamic_frame.from_catalog(database = args['db_name'], table_name = tableName, transformation_ctx = "datasource")
```

 <font color=##00aa00>2. 数据结构转换
 

```python
mapped_readings = ApplyMapping.apply(frame = datasource, mappings = [("lclid", "string", "meter_id", "string"), \
                                                                     ("datetime", "string", "reading_time", "string"), \
                                                                     ("KWH/hh (per half hour)", "double", "reading_value", "double")], \
                                     transformation_ctx = "mapped_readings")
```

 <font color=##00aa00>3. 数据结构拆分、定义
 

```python
mapped_readings_df = DynamicFrame.toDF(mapped_readings)

mapped_readings_df = mapped_readings_df.withColumn("obis_code", lit(""))
mapped_readings_df = mapped_readings_df.withColumn("reading_type", lit("INT"))

reading_time = to_timestamp(col("reading_time"), "yyyy-MM-dd HH:mm:ss")
mapped_readings_df = mapped_readings_df \
    .withColumn("week_of_year", weekofyear(reading_time)) \
    .withColumn("date_str", regexp_replace(col("reading_time").substr(1,10), "-", "")) \
    .withColumn("day_of_month", dayofmonth(reading_time)) \
    .withColumn("month", month(reading_time)) \
    .withColumn("year", year(reading_time)) \
    .withColumn("hour", hour(reading_time)) \
    .withColumn("minute", minute(reading_time)) \
    .withColumn("reading_date_time", reading_time) \
    .drop("reading_time")
```

<font color=##00aa00> 4. 清洗后的数据写入新s3
 

```python
# write data to S3
filteredMeterReads = DynamicFrame.fromDF(mapped_readings_df, glueContext, "filteredMeterReads")

s3_clean_path = "s3://" + args['clean_data_bucket']

glueContext.write_dynamic_frame.from_options(
    frame = filteredMeterReads,
    connection_type = "s3",
    connection_options = {"path": s3_clean_path},
    format = "parquet",
    transformation_ctx = "s3CleanDatasink")
```
<br>

**运行作业**
<br>
<font color="#ff0000">&nbsp;&nbsp;&nbsp;&nbsp;执行成功后，状态将变为"SUCCESS"，失败将会给出失败信息，可在CloudWatch 中查看详情

![](https://img-blog.csdnimg.cn/c8a534c6a2cf41fa9df2cdbbbb0553c5.png)

![](https://img-blog.csdnimg.cn/7786ba66e3ac430c9d0efd540a1b2bac.png)
<br>
<font color=##00aa00>清洗后的数据保存到了s3
<br>
![](https://img-blog.csdnimg.cn/d98cac38d99d4b1b82b9368c2357c98c.png)
**数据清洗完毕后，可通过上一篇中的爬网程序步骤，将清洗后的数据的结构创建表到数据目录中，
此时我们可以使用Athena对清洗后的数据进行分析。**
## 2、数据分区
接下来我们对数据进行分区处理（<font color="#ff0000">此处只提供了按天分区</font>）
重新进行数据清洗中的创建Job操作后，重写脚本
**编辑脚本**
连接数据源。表为上一步最后重新爬取生成的新表。
```python
cleanedMeterDataSource = glueContext.create_dynamic_frame.from_catalog(database = args['db_name'], table_name = tableName, transformation_ctx = "cleanedMeterDataSource")
```
根据type与data_str分区

```python
business_zone_bucket_path_daily = "s3://{}/daily".format(args['business_zone_bucket'])

businessZone = glueContext.write_dynamic_frame.from_options(frame = cleanedMeterDataSource, \
    connection_type = "s3", \
    connection_options = {"path": business_zone_bucket_path_daily, "partitionKeys": ["reading_type", "date_str"]},\
    format = "parquet", \
    transformation_ctx = "businessZone")
```
分区后的数据结果：
![](https://img-blog.csdnimg.cn/e20989d5745042c1ba4a6cfa1b67936b.png)
再次创建、运行爬网程序，将会在数据目录中生成新的分区表。

## 3、总结

&nbsp;&nbsp;&nbsp;&nbsp;到这一步，我们已经使用Glue ETL对s3桶中的数据进行了清洗、分区操作。在进行上篇中的Athena操作后，我们已经可以通过Athena直接查询到清洗、分区后的数据集了。
&nbsp;&nbsp;&nbsp;&nbsp;接下来，我们会通过使用APIGateway+Lambda+Athena来构建一个无服务器的数据查询分析服务。
