---
title: Flask框架中自定义模型类的表名、父类相关问题分析
tags: 
- python
- flask
categories:
- python
date: 2020-04-09 16:07:52
---

## 关于继承父类model，为子model自定义表名的设置

## 1、单张表设置表名

\__tablename__ = 'model name'
此字段必须是继承自models.Model才可设置。
![](https://img-blog.csdnimg.cn/20200409155726755.png)

## 2、有继承关系的表设置表名

如果其他model继承自BaseModel，那么__tablename__将失效，默认表名为父类表名。
要想自定义子类表名，需要在父类model中添加**\__abstract__ = True**字段。
![](https://img-blog.csdnimg.cn/20200409160213276.png)
![](https://img-blog.csdnimg.cn/20200409160229896.png)
这样就可以自定义表名啦。

## 3、喜欢的拿走
![](https://img-blog.csdnimg.cn/20200409160438194.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtamYx,size_16,color_FFFFFF,t_70)
