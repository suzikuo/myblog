---
title: django rest_framework认证梳理
tags: 
- python
- django
categories:
- python
date: 2020-11-29 17:07:06
---


请主人通过self.dispatch()入口查看源码。
看了一大堆，源码最清楚！！！！
1、
![使用](https://img-blog.csdnimg.cn/20201129165015745.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtamYx,size_16,color_FFFFFF,t_70)
2、流程：
![](https://img-blog.csdnimg.cn/20201129165117920.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtamYx,size_16,color_FFFFFF,t_70)
	1、入口：self.dispatch()
	2、对request进行封装：
		

```python
request = self.initialize_request(request, *args, **kwargs)
        self.request = request
```
        获取定义的认证类（全局/局部），通过列表生成式创建对象进行认证
        如果未定义认证类列表，则使用默认的两个认证类
   

```python
    def initialize_request(self, request, *args, **kwargs):
        """
        Returns the initial request object.
        """
        parser_context = self.get_parser_context(request)

        return Request(
            request,
            parsers=self.get_parsers(),
            authenticators=self.get_authenticators(),#认证
            negotiator=self.get_content_negotiator(),
            parser_context=parser_context
        )


def get_authenticators(self):
"""
Instantiates and returns the list of authenticators that this view can use.
"""

	return [auth() for auth in self.authentication_classes]


#默认认证列表
authentication_classes = api_settings.DEFAULT_AUTHENTICATION_CLASSES
```
3、对request进行加工后，进行认证
#遍历认证列表。如果子类中没有，则使用父类的默认的认证类。
       

```python
  'DEFAULT_AUTHENTICATION_CLASSES': ['rest_framework.authentication.SessionAuthentication',
         'rest_framework.authentication.BasicAuthentication']
```

,认证抛异常之前设置self.user为匿名用户，最后再抛异常。
            self.dispatch()接受异常，返回

self.initial(request, *args, **kwargs)
	&nbsp;&nbsp;&nbsp;&nbsp; -self.perform_authentication(request)
	&nbsp;&nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; -request.user(内部循环认证)
	#如果第一个认证没有通过，并且抛异常，直接返回。
        #如果第一个认证没有通过，并且返回None，则继续下一个认证
        #第一个认证通过，直接退出
        #认证未通过，或者抛异常，调用self._not_authenticated()，设置匿名用户，并且抛出异常，接着self.dispatch()捕获异常


