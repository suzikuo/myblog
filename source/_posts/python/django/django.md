---
title: django
tags: 
- python
- django
categories:
- python
date: 2021-12-29 14:46:12
---

- [一、django简](#一django简)
- [二、具体内容](#二具体内容)
- [3、DRF](#3drf)


## 一、django简

**1、组件**
	

 1. Project：项目
 2. Apps：应用。尽量减少应用与工程、应用于应用之间的依赖关系，做到功能独立。
 3. Model：ORM。对象关系映射。
 4. URL Route：URL分配器、路由。
 5. View：视图。业务。
 6. DTL：模板语言，与jinja2相同。
 7. Admin：管理界面
 8. Cache System：缓存系统。

**2、目录结构**
![](https://img-blog.csdnimg.cn/e95d7ccfa907429a95d2f0279b0ca2a2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
**3、常用命令**

 1. 新建一个 django project：`django-admin.py startproject project_name`
 2.  新建 app：`python manage.py startapp app_name`
 3. 创建数据库表 或 更改数据库表或字段：`python manage.py makemigrations（生成迁移文件）`，`python manage.py migrate（执行迁移）`
 4. 启动调试：`python manage.py runserver`、`python manage.py runserver 0.0.0.0:8000`
 5. 创建超级管理员：`python manage.py createsuperuser`、`python manage.py changepassword username（修改密码）`

## 二、具体内容

 **1. setting.py**

 **2. 请求生命周期**
![](https://img-blog.csdnimg.cn/bc326173c8eb43979d2782628ec0c153.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)

 	
 **3. 中间件**

> 中间件是一个用来处理Django请求和响应的框架级钩子。它是一个轻量、低级别的插件系统，用于在全局范围内改变Django的输入和输出。每个中间件组件都负责做一些特定的功能。
 **3. 中间件的五种方法**
>  process_request : 请求进来时,权限认证 
>  process_view : 路由匹配之后,能够得到视图函数
> process_exception : 异常时执行 
> process_template_responseprocess : 模板渲染时执行
> process_response : 请求有响应时执行

 **4. session**
> 
> django的session存储可以利用中间件来实现。需要在 settings.py 文件中注册APP、设置中间件用于启动。设置存储模式（数据库/缓存/混合存储）和配置数据库缓存用于存储，生成django_session表单用于读写。

 **5. csrf实现机制**
 

> django第1次响应来自某个客户端的请求时,服务器随机产生1个token值，把这个token保存在session中;
>  
> 同时服务器把这个token放到cookie中交给前端页面；
>  
>该客户端再次发起请求时，把这个token值加入到请求数据或者头信息中,一起传给服务器；
服务器校验前端请求带过来的token和session里的token是否一致。

 **6. 缓存**
	

> 1. **全站缓存，较少使用**

```python
MIDDLEWARE_CLASSES = (
    ‘django.middleware.cache.UpdateCacheMiddleware’,  # 第一
    'django.middleware.common.CommonMiddleware',
    ‘django.middleware.cache.FetchFromCacheMiddleware’,  # 最后
)

```

>2. **视图缓存，用户视图函数或视图类中**

```python
from django.views.decorators.cache import cache_page
import time

@cache_page(15) #超时时间为15秒
def index(request):
    t=time.time() #获取当前时间
    return render(request,"index.html",locals())

```

> **3. 模板缓存，指缓存不经常变换的模板片段**

```python
{% load cache %}
    <h3 style="color: green">不缓存:-----{{ t }}</h3>

{% cache 2 'name' %} # 存的key
    <h3>缓存:-----:{{ t }}</h3>
{% endcache %}

```

 **7. ORM操作QuerySet对象的方法**


| 方法          | 作用                                                                                                     |
| ------------- | -------------------------------------------------------------------------------------------------------- |
| all()         | 查询所有结果                                                                                             |
| filter()      | 过滤查询对象。获取不到返回None。                                                                         |
| get()         | 返回与所给筛选条件相匹配的对象，返回结果有且只有1个。如果符合筛选条件的对象超过1个或者没有都会抛出错误。 |
| exclude()     | 排除满足条件的对象                                                                                       |
| order_by()    | 对查询结果排序                                                                                           |
| reverse()     | 对查询结果反向排序                                                                                       |
| count()       | 返回数据库中匹配查询(QuerySet)的对象数量。                                                               |
| first()       | 返回第一条记录                                                                                           |
| last()        | 返回最后一条记录                                                                                         |
| exists()      | 如果QuerySet包含数据，就返回True，否则返回False                                                          |
| values()      | 返回包含对象具体值的字典的QuerySet                                                                       |
| values_list() | 与values()类似，只是返回的是元组而不是字典。                                                             |
| distinct()    | 对查询集去重                                                                                             |


**django filter过滤器中支持命令参数**
__exact 精确等于 like 'aaa'
 __iexact 精确等于 忽略大小写 ilike 'aaa'
 __contains 包含 like '%aaa%'
 __icontains 包含 忽略大小写 ilike '%aaa%'，但是对于sqlite来说，contains的作用效果等同于icontains。
__gt 大于
__gte 大于等于
__lt 小于
__lte 小于等于
__in 存在于一个list范围内
__startswith 以...开头
__istartswith 以...开头 忽略大小写
__endswith 以...结尾
__iendswith 以...结尾，忽略大小写
__range 在...范围内
__year 日期字段的年份
__month 日期字段的月份
__day 日期字段的日
__isnull=True/False

 **8. Q和F**

> Q查询：对数据的多个字段联合查询（常和且或非"&|~"进行联合使用）
F查询：对数据的不同字段进行比较（常用于比较和更新，对数据进行加减操作）


```python
Q案例：
查询作者名字是猎虎或者价格大于5000的书 -- 或
res = models.Bbook.objects.filter(Q(authors__name='猎虎')|Q(price__gt=5000))

查询作者名字是猎虎并且价格大于5000的书 -- 与 &
res = models.Bbook.objects.filter(Q(authors__name='猎虎',price__gt=5000))
查询作者名字不是猎虎的书
res = models.Bbook.objects.filter(~Q(authors__name='猎虎'))

F案例：
所有阅读数+1
res = .Bbook.objects.all().update(read_num=F('read_num')+1)
查询评论数大于阅读数的书籍
res = models.Bbook.objects.all().filter(commit_num__gt=F('read_num'))

```

 **9. 视图响应**
	

```python
return Response({content=响应体, content_type=响应体数据类型, status=状态码)
return HttpResponse(content=响应体, content_type=响应体数据类型, status=状态码) 
return JsonResponse({‘city’: ‘beijing’, ‘subject’: ‘python’},status=response.status_code)
return redirect(‘/index.html’)
```

 **10. 生命周期**
 

> 请求-->wsgi-->中间件-->路由匹配-->视图-->（DRF：通过self.dispatch()反射执行对应方法（在其中进行request的封装、权限的绑定和认证：认证、权限、节流）-->中间件-->wsgi

 **11. Celery**

> 分布式任务队列，其本质是生产者消费者模型，生产者发送任务到消息队列，消费者负责处理任务。
![](https://img-blog.csdnimg.cn/3cfba3d9849a45088bdee410a0c8ed04.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_18,color_FFFFFF,t_70,g_se,x_16)

 **12. CBV、FBV**
 

> FBV（function base views） 基于函数的视图，就是在视图里使用函数处理请求。
CBV（class base views） 基于类的视图，就是在视图里使用类处理请求。

FBV：

```python
urlpatterns = [
    path("login/", views.login),
]

from django.shortcuts import render,HttpResponse

def login(request):
    if request.method == "GET":
        return HttpResponse("GET 方法")
    if request.method == "POST":
        user = request.POST.get("user")
        pwd = request.POST.get("pwd")
        if user == "runoob" and pwd == "123456":
            return HttpResponse("POST 方法")
        else:
            return HttpResponse("POST 方法1")
```
CBV：

```python
urlpatterns = [
    path("login/", views.Login.as_view()),
]

from django.shortcuts import render,HttpResponse
from django.views import View

class Login(View):
    def get(self,request):
        return HttpResponse("GET 方法")

    def post(self,request):
        user = request.POST.get("user")
        pwd = request.POST.get("pwd")
        if user == "runoob" and pwd == "123456":
            return HttpResponse("POST 方法")
        else:
            return HttpResponse("POST 方法 1")
```

 **13. 连接多数据库**
 在 settings.py 中配置需要连接的多个数据库连接串
 

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    },
    'ora1': {   # 配置第二个数据库节点名称
        'ENGINE': 'django.db.backends.oracle',
        'NAME': 'devdb',
        'USER': 'hysh',
        'PASSWORD': 'hysh',
        'HOST': '192.168.191.3',
        'PORT': '1521',
    },
}

DATABASE_ROUTERS = ['Prject.database_router.DatabaseAppsRouter']

设置APP对应的数据库路由表 
DATABASE_APPS_MAPPING = {
    # example:
    # 'app_name':'database_name',
    'report': 'ora1',
    'admin': 'defualt',
    'regs':  'defualt',
}
```
创建数据库路由规则 

```python
from django.conf import settings
 
DATABASE_MAPPING = settings.DATABASE_APPS_MAPPING
 
 
class DatabaseAppsRouter(object):
 
    def db_for_read(self, model, **hints):
        if model._meta.app_label in DATABASE_MAPPING:
            return DATABASE_MAPPING[model._meta.app_label]
        return None
 
    def db_for_write(self, model, **hints):
        if model._meta.app_label in DATABASE_MAPPING:
            return DATABASE_MAPPING[model._meta.app_label]
        return None
 
    def allow_relation(self, obj1, obj2, **hints):
        """Allow any relation between apps that use the same database."""
        db_obj1 = DATABASE_MAPPING.get(obj1._meta.app_label)
        db_obj2 = DATABASE_MAPPING.get(obj2._meta.app_label)
        if db_obj1 and db_obj2:
            if db_obj1 == db_obj2:
                return True
            else:
                return False
        return None
 
    def allow_syncdb(self, db, model):
        """Make sure that apps only appear in the related database."""
 
        if db in DATABASE_MAPPING.values():
            return DATABASE_MAPPING.get(model._meta.app_label) == db
        elif model._meta.app_label in DATABASE_MAPPING:
            return False
        return None
 
    def allow_migrate(self, db, app_label, model=None, **hints):
        """
        Make sure the auth app only appears in the 'auth_db'
        database.
        """
        if db in DATABASE_MAPPING.values():
            return DATABASE_MAPPING.get(app_label) == db
        elif app_label in DATABASE_MAPPING:
            return False
        return None
```
model

```python
class Users(models.Model):
    name = models.CharField(max_length=50)
    passwd = models.CharField(max_length=100)
 
    def __str__(self):
        return "app01 %s " % self.name
 
    class Meta:
        app_label = "app01"
```

一个app使用多个数据库

```python
models.User.objects.using(dbname).all(） 
```

## 3、DRF
参考文章：[添加链接描述](https://blog.csdn.net/qq_41500222/article/details/87895643)
![](https://img-blog.csdnimg.cn/6128f9a7523f4d50acc49c39fa957d51.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_10,color_FFFFFF,t_70,g_se,x_16)
所有的类所多继承的父类必须是APIView
可继承的类：

```python
#class View(object):
#class APIView(View): 封装了view,并且重新封装了request,初始化了各种组件
#class GenericAPIView(views.APIView):
#1.增加了一些属性和方法,如get_queryset,get_serializer
#class GenericViewSet(ViewSetMixin, generics.GenericAPIView)
#父类ViewSetMixin 重写了as_view,返回return csrf_exempt(view)
#并重新设置请求方式与执行函数的关系
class ModelViewSet(mixins.CreateModelMixin,
                   mixins.RetrieveModelMixin,
                   mixins.UpdateModelMixin,
                   mixins.DestroyModelMixin,
                   mixins.ListModelMixin,
                   GenericViewSet):pass
#继承了mixins下的一些类,封装了list,create,update等方法
#和GenericViewSet
```

