---
title: Django 全局获取request
tags: 
- python
- django
categories:
- python
date: 2023-03-14 17:09:45
---

- [前言](#前言)
- [一、能否直接pip安装？](#一能否直接pip安装)
- [二、如何使用？](#二如何使用)
  - [1.引入](#1引入)
  - [2.使用](#2使用)
- [三、如何实现的？](#三如何实现的)
  - [1、django Middleware原理](#1django-middleware原理)
  - [2、中间件是线程隔离的吗？](#2中间件是线程隔离的吗)
  - [3、django\_middleware\_global\_request实现原理](#3django_middleware_global_request实现原理)
    - [3.1 GlobalRequestMiddleware调用顺序](#31-globalrequestmiddleware调用顺序)
    - [3.2 如何实现线程隔离的？](#32-如何实现线程隔离的)
  - [参考文献](#参考文献)


# 前言


Django项目中使用到了Singal来进行操作日志的记录的操作，但需要在Signal中获取reqeust中的某些数据。因此，如果有一个全局的request，将会对之后的操作带来很大的方便。

---


# 一、能否直接pip安装？
django_middleware_global_request
```python
pip install django-middleware-global-request
```

# 二、如何使用？
## 1.引入
在项目的setting.py文件的MIDDLEWARE添加中间件。
![](https://img-blog.csdnimg.cn/d9c124e640a34ac7b8183b2f684cd581.png)



## 2.使用
代码如下（示例）：

```python
message_dict["args"]["user_id"] = get_request().user.id
```


# 三、如何实现的？
## 1、django Middleware原理

django在初始化时，会先加载middleware。

```python
class WSGIHandler(base.BaseHandler):
    request_class = WSGIRequest

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.load_middleware()
    # 子线程中调用
    def __call__(self, environ, start_response):
        request = self.request_class(environ)
        response = self.get_response(request)
```

```python
class BaseHandler:
    def load_middleware(self, is_async=False):
            get_response = self._get_response_async if is_async else self._get_response
        	handler = convert_exception_to_response(get_response)
        	# 初始handler为Basehandler的_get_response方法。 
            for middleware_path in reversed(settings.MIDDLEWARE):
            	# 我们每个中间件
            	middleware = import_string(middleware_path)
            	# 初始adapted_handler 为Basehandler的_get_response方法。
            	adapted_handler = self.adapt_method_mode(
                    middleware_is_async, handler, handler_is_async,
                    debug=settings.DEBUG, name='middleware %s' % middleware_path,
                )
            	mw_instance = middleware(adapted_handler)
            	......
            	handler = adapted_handler
            	......
            	# 记录每个中间件的流程方法.
	            if hasattr(mw_instance, 'process_view'):
	                self._view_middleware.insert(...)
	            if hasattr(mw_instance, 'process_template_response'):
	                self._template_response_middleware.append(...)
	            if hasattr(mw_instance, 'process_exception'):
	                self._exception_middleware.append(...)
	        ......
	        self._middleware_chain = handler
```
在初始化时，会实例化每个middleware类。等到for循环结束时，`handler = A(B(C(D(_get_response))))`，将每个中间件层层嵌套起来。
在此过程中，会将中间件的`process_view` `process_template_response ` `process_exception` 存储到列表中等待调用。
在     `get_response(self, request)`中调用,`get_response(self, request)`在子线程中被调用。

```python
    def get_response(self, request):
    	...
        response = self._middleware_chain(request)
        ....
        return response
```
`self._middleware_chain(request)` 会层层调用中间件的`__call__`方法。
等到所有的中间件都调用完成之后，将调用`BaseHandler._get_response`方法，`A(B(C(D(_get_response))))`

```python

    def _get_response(self, request):
    		# 便利调用所有中间件的process_view方法
            for middleware_method in self._view_middleware:
            	response = middleware_method(request, callback, callback_args, callback_kwargs)
            	if response:
            	    break
           	...
           	# 便利调用所有中间件的process_template_response 方法
           	if hasattr(response, 'render') and callable(response.render):
           		for middleware_method in self._template_response_middleware:
           			response = middleware_method(request, response)
            try:
                response = response.render()
            except Exception as e:
            	# 便利调用所有中间件的process_exception方法
                response = self.process_exception_by_middleware(e, request)
                if response is None:
                    raise
```
在_get_response中，会调用中间件的`process_view` `process_template_response ` `process_exception` 

再看一下`MiddlewareMixin`的调用

```python
class MiddlewareMixin:
    def __call__(self, request):
		......
        response = None
        if hasattr(self, 'process_request'):
            response = self.process_request(request)
        response = response or self.get_response(request)
        if hasattr(self, 'process_response'):
            response = self.process_response(request, response)
        return response
```
`MiddlewareMixin.__call__`在上面被调用。
可以看到，中间件的执行顺序为：`process_request`>`process_view`>`process_template_response (看情况)`>`process_exception(有异常)`>`process_response`

## 2、中间件是线程隔离的吗？
通过上面的代码，我们能看到，中间件只会在项目初始化时，有WSGIHandler初始化一次。并不是每个线程进来都会初始化。
因此需要解决线程冲突问题。


## 3、django_middleware_global_request实现原理
### 3.1 GlobalRequestMiddleware调用顺序

```python
class GlobalRequestMiddleware(object):

    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        with GlobalRequest(request=request):
            return self.get_response(request)
```

```python
class GlobalRequestStorage(object):
    storage = threading.local()

    def get(self):
        if hasattr(self.storage, "request"):
            return self.storage.request
        else:
            return None

    def get_user(self, request=None):
        request = request or self.get()
        return getattr(request, "user", None)

    def set(self, request):
        self.storage.request = request

    def set_user(self, user, request=None):
        if request:
            self.storage.request = request
        if not hasattr(self.storage, "request"):
            self.storage.request = HttpRequest()
        if user:
            self.storage.request.user = user

    def recover(self, request=None, user=None):
        if hasattr(self.storage, "request"):
            del self.storage.request
        if request:
            self.storage.request = request
            if user:
                self.storage.request.user = user
```

```python
class GlobalRequest(object):

    def __init__(self, request=None, user=None):
        self.global_request_storage = GlobalRequestStorage()
        self.new_request = request or HttpRequest()
        self.new_user = user
        self.old_request = self.global_request_storage.get()
        self.old_user = self.global_request_storage.get_user(self.old_request)

    def __enter__(self):
        self.global_request_storage.set_user(user=self.new_user, request=self.new_request)
        return self.global_request_storage.get()

    def __exit__(self, *args, **kwargs):
        self.global_request_storage.recover(request=self.old_request, user=self.old_user)

```

`GlobalRequest` 使用上下文管理request。
在`__init__`时，实例化`GlobalRequestStorage`
在`__enter__` 时，将当前的request存储到`GlobalRequestStorage()`中，
在`__exit__`时，将`GlobalRequestStorage`中`del self.storage.request`
### 3.2 如何实现线程隔离的？
线程隔离：`threading.local()`

    threading.local的作用：为每个线程开辟一块空间进行数据的存储
    空间与空间之间数据是隔离的
类似于Flask框架的实现。Flask框架也是借用了`threading.local()`在处理线程隔离，Flask中维护了一个字典，来存储app、request、g等。

```python
def get_request():
    return GlobalRequestStorage().get()
```
`get_request()`中又新实例化了一个`GlobalRequestStorage`，但`storage = threading.local()`将会从当前线程的数据块中获取数据。

---


## 参考文献
[threading.local](https://blog.csdn.net/weixin_42289273/article/details/116831590)
[Django系列之中间件加载原理](https://blog.csdn.net/weixin_30823683/article/details/96553451)
