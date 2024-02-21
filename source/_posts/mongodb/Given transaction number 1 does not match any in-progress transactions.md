---
title: Given transaction number 1 does not match any in-progress transactions
tags: 
- python
- mongodb
categories:
- python
date: 2021-12-28 14:59:27
---

**错误信息：**
**Given transaction number 1 does not match any in-progress transactions**
**出错代码：**

```python
getattr(self._mg_db_write, PublicationCollectionInfoTable.table_name).find_one_and_update(query_dict, update_dict,upsert=True,session=session)
```

在使用pymongo 操作mongodb时，使用了事务来进行数据插入与更新
![](https://img-blog.csdnimg.cn/4897072ad2034d948d09c456aac87785.png)
![](https://img-blog.csdnimg.cn/117c41cefe044909a6dba66425bb293f.png)
在本地测试没有问题，在线上环境使用报错：
`Given transaction number 1 does not match any in-progress transactions`
**原因**：
主要原因：数据库中未创建表！
1、mongodb版本问题，线上与线下不统一。
线上版本低于线下版本，线上的mongodb.find_one_and_update暂不支持session共享，但insert_one 和 update_one 支持共享session。
2、session超时，默认事务生命周期最大值为1分钟
**目前解决方案：**
解决方案：手动新建对应表
1、统一mongodb版本
2、将find_one_and_update拆为find_one和insert_one或update_one
3、手动回滚：获取操作成功的数据，在某一步出错时，手动还原所有操作。如insert的delete掉。
4、session超时解决自己百度
