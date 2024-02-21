---
title: django DRF 组件解析
tags: 
- python
- django
categories:
- python
date: 2022-07-06 16:09:58
---

- [一、认证](#一认证)
          - [1、在class类里面的(APIView)在pycharm按住ctrl点击APIView,然后向下翻找到](#1在class类里面的apiview在pycharm按住ctrl点击apiview然后向下翻找到)
- [二、权限](#二权限)
- [三、限流](#三限流)
- [四、版本](#四版本)
- [五、解析器(parser)](#五解析器parser)
- [六、序列化器](#六序列化器)


# 一、认证

###### 1、在class类里面的(APIView)在pycharm按住ctrl点击APIView,然后向下翻找到![](https://img-blog.csdnimg.cn/5403b26c0ae04508b95d196d35a9cc52.png)

```python
     def dispatch(self, request, *args, **kwargs):
        """
        `.dispatch()` is pretty much the same as Django's regular dispatch,
        but with extra hooks for startup, finalize, and exception handling.
        """
        self.args = args
        self.kwargs = kwargs
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        self.headers = self.default_response_headers  # deprecate?


        try:
            #进这里 #点击这个
            self.initial(request, *args, **kwargs)

            # Get the appropriate handler method
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed

            response = handler(request, *args, **kwargs)

        except Exception as exc:
            response = self.handle_exception(exc)

        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response


```
找到def dispatch(self, request, *args, **kwargs):中的 self.initial并点击

```python
        try:
           #进这里 #点击这个
           self.initial(request, *args, **kwargs)

```
然后将会看到:
![](https://img-blog.csdnimg.cn/84e34c7cd6284716bacf8f0f2d8f5f16.png)

```python
		# 代表 认证组件
		self.perform_authentication(request)
		# 代表 权限组件
        self.check_permissions(request)
        # 代表 限流组件
        self.check_throttles(request)

```
然后 点击self.perform_authentication(request)这个方法
![](https://img-blog.csdnimg.cn/9613fb4c0e714b4cb3011c9af6ac7699.png)
在这里它封装了 request.user方法 点击request.user然后选择第三个
![](https://img-blog.csdnimg.cn/93f394b960de45d7a35add1d49f3af59.png)
在这里封装了self._authenticate() 点击self._authenticate()
![](https://img-blog.csdnimg.cn/8a62770a3ce24b68892b25ddc848c276.png)
![](https://img-blog.csdnimg.cn/d20aa51fec7b4aee87d6fb67e2273322.png)
# 二、权限
首先点击 self.dispatch() 找到 self.initial(request,*args,**kwargs)点击 进入
![](https://img-blog.csdnimg.cn/499dd6473f124ca0bdb899bd55af156f.png)
![](https://img-blog.csdnimg.cn/571886f93a3a410596e0ff2ed4c7361f.png)
然后点击 代表权限的组件方法
这里会出现 点击这个选择第三个 会出现我们冲的的方法
![](https://img-blog.csdnimg.cn/a712f7b30ea34c338390d8b86f13ac90.png)
随后滚动鼠标 会出现这个方法
![](https://img-blog.csdnimg.cn/bb51397df03648eea823fe042b0d39c2.png)
我们在settings.py设置全局组件配置 权限 首先我们要找到源码来写入 这个配置的功能
在class类里面的(APIView)在pycharm按住ctrl点击APIView,然后向下翻找到
![](https://img-blog.csdnimg.cn/65143158615b42678bcaf912caf87427.png)
DEFAULT_PERMISSION_CLASSES 这个就代表配置 settings.py需要设置的方法 [这里写的是方法类的路径]
![](https://img-blog.csdnimg.cn/45d14baa1abd4e6e9274e2b94623dd45.png)
然后在view.py订单视图中 写入

```python
class OrderView(APIView):
    """
    订单相关业务
    """
    authentication_classes = []
    permission_classes = []

```
# 三、限流
限流就是限制访问频率或者次数
我们可以看看源码是怎样写的
首先点击 self.dispatch() 找到 self.initial(request,*args,**kwargs)点击 进入
![](https://img-blog.csdnimg.cn/375ca3e8195949a39bd7c30ce94d3bd5.png)
然后点击 代表限流的组件方法
![](https://img-blog.csdnimg.cn/bbce45e5fb7a4b08a897cf53058af5ec.png)
在这里会找到封装的allow_request 点击它 选择第三个
![](https://img-blog.csdnimg.cn/087fbb0c6b624ad9a1bf297851dac5df.png)
这里就会出现allow_request源码的方法 和 wait源码的方法
![](https://img-blog.csdnimg.cn/895ba1f17ff748908555278f0bb4aa9a.png)
![](https://img-blog.csdnimg.cn/359bb67fb2f04749aca3221c34f24ff3.png)
我们开始在views.py设置 一个类来测试我们书写的代码逻辑是否正确

```python
# 在我的组件包导入 MyThrottle方法
from myapp.myutils.throttle import MyThrottle


class StudentView(APIView):
    authentication_classes = [MyAuthtication]
    permission_classes = [SVIPPermission]

    # 开启这个代表自己设置的限流
    throttle_classes = [MyThrottle]

	
    def get(self, request, *args, **kwargs):
    	# 获取当前的ip
        ip = request.META.get('REMOTE_ADDR')
        # 并返回当前的ip
        return JsonResponse({'ip': ip})

```
也可以在settings.py全局设置组件
![](https://img-blog.csdnimg.cn/fd4699b9405a4eca8a296ce554482636.png)
怎么找到它呢 ?
在class类里面的(APIView)在pycharm按住ctrl点击APIView,然后向下翻找到
![](https://img-blog.csdnimg.cn/ec6a94e9a8e545c69dbbb96da3eb94a2.png)
settings.py全局设置
![](https://img-blog.csdnimg.cn/3f614336c28d4461b3d6dfaeb384d19e.png)
![](https://img-blog.csdnimg.cn/898dbb4b9345407eaf023f0ea32a4203.png)
找到这个 这里是相关的一些配置
![](https://img-blog.csdnimg.cn/aff0e7adced549aab4e996cdf23c475d.png)
![](https://img-blog.csdnimg.cn/5129927f84c047099a2fb408c324a165.png)
这个是使用的方法
![](https://img-blog.csdnimg.cn/98543b932bcd4f4d9b55a00a30b8250c.png)
# 四、版本
![](https://img-blog.csdnimg.cn/a7b2c04f28ce43dd8734995e3782dfd8.png)
一种是原生的源码

```python
# QueryParameterVersioning点击这个里面会有个这种方法
from rest_framework.versioning import QueryParameterVersioning

```
点击这个
![](https://img-blog.csdnimg.cn/0fbd44d53f6a462ba712072512fb4fe4.png)
![](https://img-blog.csdnimg.cn/beb4571440ff4799b079a591aa9d1804.png)
含义
![](https://img-blog.csdnimg.cn/b29f826a623b4d7fa1077c95089b723a.png)
![](https://img-blog.csdnimg.cn/64c9d5af7bb34b468e9cf99bac4ebcd0.png)
![](https://img-blog.csdnimg.cn/4207f1b293454c6b82ba8a468bdaf2aa.png)
点击这个选择第三个
![](https://img-blog.csdnimg.cn/bb831c44ff724d31b1ea575d7be2bfc8.png)
![](https://img-blog.csdnimg.cn/e0849f7ffbad4f28a6ed0266417c3d80.png)
![](https://img-blog.csdnimg.cn/05cfe6d2d1ff485f9c9e15498e62de3e.png)
复制这个到settings.py进行全局配置
![](https://img-blog.csdnimg.cn/5dcef4b8c0534d29bb114587d84d3705.png)
![](https://img-blog.csdnimg.cn/ece96aa6de4f40ad9da748f981e8af5b.png)

```python
REST_FRAMEWORK = {
    "DEFAULT_VERSION": "v1",# 默认版本
    "ALLOWED_VERSIONS": ["v1", "v2"],# 被允许访问的版本

}

```
**另一种方法**

```python
# 从rest_framework.versioning 导入 URLPathVersioning
from rest_framework.versioning import URLPathVersioning

# 设计一个类
class UsersView(APIView):
    versioning_class = URLPathVersioning  # 调入版本这个方法

    def get(self, request, *args, **kwargs):
    	# 获取版本
        v = request.version
        u1 = request.versioning_scheme.reverse(viewname="user",request=request)
        print(u1)
        return HttpResponse(v)

```
点击这个URLPathVersioning看源码

```python
from rest_framework.versioning import URLPathVersioning

```
继承的是这个方法点击
![](https://img-blog.csdnimg.cn/3186f2e59f5c46fa89a1eebed780115a.png)
鼠标滚动找到复制这个 放到urls.py路由中
![](https://img-blog.csdnimg.cn/17530b78373445519333e717515a2c39.png)
# 五、解析器(parser)
![](https://img-blog.csdnimg.cn/3c4ca87c6e574be98832f3c0a1c70b19.png)
![](https://img-blog.csdnimg.cn/a6d8ebc87e894252ad30b5aa60068e4f.png)
![](https://img-blog.csdnimg.cn/578cf07560fa46c28007c3e7381446d8.png)
![](https://img-blog.csdnimg.cn/e8880570969140869140bf85a0835947.png)
![](https://img-blog.csdnimg.cn/4f97d50d6c4448319f40b9c845a2f22a.png)
![](https://img-blog.csdnimg.cn/33180d1749ac4c1ab0281536e83301d2.png)
![](https://img-blog.csdnimg.cn/0755a098866947c2a59589cdadcd4db8.png)
![](https://img-blog.csdnimg.cn/2d15592a688344df96d6d2987b4c1576.png)
![](https://img-blog.csdnimg.cn/2b47b76026d74b57b932d8be04b85410.png)
# 六、序列化器
![](https://img-blog.csdnimg.cn/3db1268561f44eb8b9f3de2b9be77fd7.png)

