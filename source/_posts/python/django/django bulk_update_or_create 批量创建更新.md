---
title: django bulk_update_or_create 批量创建更新
tags: 
- python
- django
categories:
- python
date: 2023-03-14 18:05:53
---

- [前言](#前言)
- [一、代码实现](#一代码实现)
- [二、使用](#二使用)


---

# 前言
当前django并没有提供能够批量创建或更新的方法，只有`bulk_update` 和`bulk_create`以及`create_or_update`，在实际业务中并不能满足我们的需求。因此才会有了`bulk_update_or_create `

---


# 一、代码实现

```python
from utils import time_tools
from django.db import models
from django.db import transaction
# from bulk_update_or_create import BulkUpdateOrCreateQuerySet


class ExpandQuerySet(models.QuerySet):
    """
    拓展querySet
    """

    def bulk_update_or_create(self, common_keys, unique_key_name, unique_key_to_defaults, batch_size=200):
        """
        common_keys: {field_name: field_value} 通用筛选条件
        unique_key_name: field_name # 唯一字段
        unique_key_to_defaults: {field_value: {field_name: field_value}} # 更新值
        """
        with transaction.atomic(using=self.db, savepoint=False):
            filter_kwargs = dict(common_keys)
            filter_kwargs[f"{unique_key_name}__in"] = unique_key_to_defaults.keys()
            existing_objs = {
                getattr(obj, unique_key_name): obj
                for obj in self.filter(**filter_kwargs).select_for_update()
            }
            # 批量创建
            create_data = {
                k: v for k, v in unique_key_to_defaults.items() if k not in existing_objs
            }
            for unique_key_value, obj in create_data.items():
                obj[unique_key_name] = unique_key_value
                obj.update(common_keys)
            creates = [self.model(**obj_data) for obj_data in create_data.values()]
            if creates:
                self.bulk_create(creates, batch_size=batch_size)
            # 如果使用了add_now来自动更新时间，update_fields必须包含此字段
            # 因queryset.update不会自动更新时间，只有save会
            update_fields = {"update_time"}
            # 批量更新
            updates = []
            for key, obj in existing_objs.items():
                for i in unique_key_to_defaults[key].items():
                    setattr(obj, i[0], i[1])
                # 将所有要更新的字段都统计出来
                update_fields.update(unique_key_to_defaults[key].keys())
                updates.append(obj)
            if existing_objs:
                self.bulk_update(updates, update_fields, batch_size=batch_size)
        return len(creates), len(updates)

    def update(self, **kwargs):
        if getattr(self.model, "update_time", None):
            kwargs.update({"update_time": time_tools.now()})
        return super().update(**kwargs)

```

```python
class AbstractModel(models.Model):
    objects: ExpandQuerySet = ExpandQuerySet.as_manager()
```
我们需要在我们的模型基类中重定义objects为我们的扩展类。
bulk_update_or_create共分为三步：1、查询数据库中包含当前数据的数据。2、调用父类批量创建`bulk_create`方法，批量创建数据库中不存在的。3、调用父类批量更新`bulk_update`方法，批量更新数据库中存在的。
ps:我们会在批量更新中定义一个`update_fields = {"update_time"}`字段，
因为如果使用了add_now来自动更新时间，query.update 不会更新此此段，只有save中会，在save中对包含add_now的字段做了处理.

# 二、使用

```python
        bulk_create_or_update_dict = {}
        for _ in self._contents:
            if "url" in _:
                unique = _["unique"]
                permission_dict = {
                    Permission.name.field.attname: 1,
                    Permission.valid.field.attname: True,
                    Permission.desc.field.attname: 1,
                    Permission.level.field.attname: 1,
                    Permission.uri.field.attname: 1,
                    Permission.lambda_name.field.attname: 1,
                    Permission.action_name.field.attname: 1,
                    Permission.permission.field.attname: unique
                }

                bulk_create_or_update_dict[unique] = permission_dict
        common_keys = {}  # 通用筛选项
        unique_key_name = Permission.permission.field.attname  # 唯一值
        unique_key_to_defaults = bulk_create_or_update_dict  # 默认值
        Permission.objects.using(PG_WRITE).bulk_update_or_create(common_keys, unique_key_name, unique_key_to_defaults)
```
`common_keys :`字典，通用的筛选项，会传递到orm的filter中
`unique_key_name :`字符串，唯一的字段的名称
`unique_key_to_defaults :`字典，key为唯一的值，value为更新的数据
