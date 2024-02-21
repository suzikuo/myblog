---
title: Starlette 源码分析
tags: 
- python
- fastapi
categories:
- python
date: 2023-08-23 14:05:40
---

# Table of Contents
- [Table of Contents](#table-of-contents)
- [1. 服务的执行流程](#1-服务的执行流程)
  - [1.1. 执行流程的源码分析](#11-执行流程的源码分析)
    - [1.1.1. Starlette](#111-starlette)
    - [1.1.2. ServerErrorMiddleware](#112-servererrormiddleware)
    - [1.1.3. ExceptionMiddleware](#113-exceptionmiddleware)
    - [1.1.4. Router](#114-router)



# 1. 服务的执行流程
Starlette 的执行包含 3 个流程

 - 服务启动时, 也就是 asgi 的 startup 事件
 - 处理用户请求时, 也就是 asgi 的 http, websocket 事件
 - 服务停止时, 也就是 asgi 的 shutdown 事件
 
3 个执行流程入口都是 starlette.application.Starlette.__call__(scope, receive, send)

> starlette.application.Starlette.__call__(scope, receive, send)
--> ServerErrorMiddleware
----> ExceptionMiddleware
------> 自定义中间件
--------> Router
----------> 业务接口函数

每一个中间件都是一个可调用对象, 并且接受 3 个参数,也就是 asgi 协议的三个参数: scope, receive, send.
## 1.1. 执行流程的源码分析
### 1.1.1. Starlette

 - 这一层是服务的入口, 包含三个作用, 服务启动时, asgi startup 事件的执行, 以及服务停止时 asgi shutdown
   事件的执行, 客户端请求的处理.
 - 上面的执行流程的构建也是在这里构建的, 也就是中间件的加载
 - 服务的路由的,嵌套路由也是在这里构建
 - 服务的异常处理配置也是在这里定义的
 - 服务的 `startup` 和 `shutdown` 事件的绑定也是在这里,但是是绑定到 `router` 对象中的


1、流程的源码

```python
async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        scope["app"] = self
        await self.middleware_stack(scope, receive, send)
```
这里可以看出他就是往 `scope` 中添加了 `app` 属性作为对象本身, 也就是说后续的调用链路中都可以通过 `scope` 拿到 `starlette` 对象.

2、**middleware_stack**
这里的 `middleware_stack` 是就是包含了最上面的执行流程, 有点类似于常见的多层次装饰器

`Starlette.middleware_stack` 属性是在 `Starlette` 对象创建时就构建了

```python
class Starlette:
    def __init__(
        # ...
        middleware: typing.Optional[typing.Sequence[Middleware]] = None,
        exception_handlers: typing.Optional[
            typing.Mapping[
                typing.Any,
                typing.Callable[
                    [Request, Exception],
                    typing.Union[Response, typing.Awaitable[Response]],
                ],
            ]
        ] = None,
        # ...
    ) -> None:
        # ...
        self.exception_handlers = (
            {} if exception_handlers is None else dict(exception_handlers)
        )
        self.user_middleware = [] if middleware is None else list(middleware)
        self.middleware_stack = self.build_middleware_stack()

    def build_middleware_stack(self) -> ASGIApp:
        debug = self.debug
        error_handler = None
        exception_handlers: typing.Dict[
            typing.Any, typing.Callable[[Request, Exception], Response]
        ] = {}

        for key, value in self.exception_handlers.items():
            if key in (500, Exception):
                error_handler = value
            else:
                exception_handlers[key] = value

        middleware = (
            [Middleware(ServerErrorMiddleware, handler=error_handler, debug=debug)]
            + self.user_middleware
            + [
                Middleware(
                    ExceptionMiddleware, handlers=exception_handlers, debug=debug
                )
            ]
        )

        app = self.router
        for cls, options in reversed(middleware):
            app = cls(app=app, **options)
        return app
```

```python
middleware = (
              [Middleware(ServerErrorMiddleware, handler=error_handler, debug=debug)]
              + self.user_middleware
              + [
                  Middleware(
                      ExceptionMiddleware, handlers=exception_handlers, debug=debug
                  )
              ]
          )
app = self.router
for cls, options in reversed(middleware):
    app = cls(app=app, **options)
return app
```
从这里可以看出 `ServerErrorMiddleware` 是最外层的调用, 并且有个 error_handler 参数, 这个参数从上面可以看出, 如果 `exception_handlers` 字典中包含了 500 这个错误码对应的异常处理器的话, `ServerMiddleware` 在执行过程中发生错误时就会使用该函数进行处理.

中间是用户自定义的中间件
`ExceptionMiddleware` 是最内第二层, 且接受了 `exception_handlers` 参数作为其调用发生异常时的各种异常情况的处理器调用

`app = self.router` 才是真正的最内一层, `router` 就是一个 `Router` 对象, 包含了所有的 Route 列表(Route 就是定义的路由)

```python
class Starlette:
    def __init__(
        self,
        debug: bool = False,
        routes: typing.Optional[typing.Sequence[BaseRoute]] = None,
        # ...
    ) -> None:
        # ...
        self.router = Router(
            routes, on_startup=on_startup, on_shutdown=on_shutdown, lifespan=lifespan
        )
        # ...
```
**Tips:**

> 1、如果希望在服务启动和停止的执行过程中加入自定义的内容, 那么久可以通过中间件结合判断 `scope['type']=='lifespan'` 来实现; 如果是想在http 或者websocket 请求处理过程中加入自定义的内容,就可以使用中间件,并且结合`scope['type']=='http'` 或者 `'websocket'` 来实现
> 		2、如果是想在服务请求过程中抛出特定异常并且能够全局补货处理, 那么可以自定义异常类, 并且在配置该异常类对应的处理函数
> 	/3、如果不希望使用Startlett 默认的中间件,那么可以继承子类, 并修改 `middlware_stack`, 实际上完全可以自己按照 `asgi` 协议开发一个完全自定义的框架, 只是无法使用现成的代码了

### 1.1.2. ServerErrorMiddleware
他对全局的执行流程进行异常捕获, 并且对异常进行处理.

```python
class ServerErrorMiddleware:
    """
    Handles returning 500 responses when a server error occurs.

    If 'debug' is set, then traceback responses will be returned,
    otherwise the designated 'handler' will be called.

    This middleware class should generally be used to wrap *everything*
    else up, so that unhandled exceptions anywhere in the stack
    always result in an appropriate 500 response.
    """

    def __init__(
        self,
        app: ASGIApp,
        handler: typing.Optional[typing.Callable] = None,
        debug: bool = False,
    ) -> None:
        self.app = app
        self.handler = handler
        self.debug = debug

    async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return

        response_started = False

        async def _send(message: Message) -> None:
            nonlocal response_started, send

            if message["type"] == "http.response.start":
                response_started = True
            await send(message)

        try:
            await self.app(scope, receive, _send)
        except Exception as exc:
            request = Request(scope)
            if self.debug:
                # In debug mode, return traceback responses.
                response = self.debug_response(request, exc)
            elif self.handler is None:
                # Use our default 500 error handler.
                response = self.error_response(request, exc)
            else:
                # Use an installed 500 error handler.
                if is_async_callable(self.handler):
                    response = await self.handler(request, exc)
                else:
                    response = await run_in_threadpool(self.handler, request, exc)

            if not response_started:
                await response(scope, receive, send)

            # We always continue to raise the exception.
            # This allows servers to log the error, or allows test clients
            # to optionally raise the error within the test case.
            raise exc
```
异常捕获在 try except 处

如果有异常发生时

 - 且当前的 `debug` 模式, 那么就会将异常的调用栈构建成网页返回给客户端 
 - 如果不是 `debug` 模式
 	 - 并且 `handler` 配置了(这个 handler 是实例化 starlette 对象时传递的 `exception_handlers` 中 500 状态码对应的 handler), 那么就用这个 handler 进行处理. 这里要注意, handler 可以是协程对象也可以是普通函数, 他们都接受两个参数(request,exc) 且需要返回一个可调用对象, 该对象接受 (scope, receive, send) 三个参数 并且需要对客户端进行响应
	- 如果没有 handler 就直接使用 `PlainTextResponse("Internal Server Error", status_code=500)(scope, receive, send)` 返回给客户端, 也就是返回一个 500 的错误, 且响应体是 "Internal Server Error"

 **Tips:** 
 

> 如果想要更改默认的服务器异常处理, 那么配置异常处理函数时,指定 500 状态码即可

### 1.1.3. ExceptionMiddleware
这个异常处理事在 Router 的上一层进行的, 主要是对 Router 下层的异常进行捕获, 并且在发生异常时对特定异常进行处理了:

```python
class ExceptionMiddleware:
    def __init__(
        self,
        app: ASGIApp,
        handlers: typing.Optional[
            typing.Mapping[typing.Any, typing.Callable[[Request, Exception], Response]]
        ] = None,
        debug: bool = False,
    ) -> None:
        self.app = app
        self.debug = debug  # TODO: We ought to handle 404 cases if debug is set.
        self._status_handlers: typing.Dict[int, typing.Callable] = {}
        self._exception_handlers: typing.Dict[
            typing.Type[Exception], typing.Callable
        ] = {
            HTTPException: self.http_exception,
            WebSocketException: self.websocket_exception,
        }
        if handlers is not None:
            for key, value in handlers.items():
                self.add_exception_handler(key, value)

    # ...

    async def __call__(self, scope: Scope, receive: Receive, send: Send) -> None:
        if scope["type"] not in ("http", "websocket"):
            await self.app(scope, receive, send)
            return

        response_started = False

        async def sender(message: Message) -> None:
            nonlocal response_started

            if message["type"] == "http.response.start":
                response_started = True
            await send(message)

        try:
            await self.app(scope, receive, sender)
        except Exception as exc:
            handler = None

            if isinstance(exc, HTTPException):
                handler = self._status_handlers.get(exc.status_code)

            if handler is None:
                handler = self._lookup_exception_handler(exc)

            if handler is None:
                raise exc

            if response_started:
                msg = "Caught handled exception, but response already started."
                raise RuntimeError(msg) from exc

            if scope["type"] == "http":
                request = Request(scope, receive=receive)
                if is_async_callable(handler):
                    response = await handler(request, exc)
                else:
                    response = await run_in_threadpool(handler, request, exc)
                await response(scope, receive, sender)
            elif scope["type"] == "websocket":
                websocket = WebSocket(scope, receive=receive, send=send)
                if is_async_callable(handler):
                    await handler(websocket, exc)
                else:
                    await run_in_threadpool(handler, websocket, exc)

    # ...
```
他本身是包含两个预置的异常处理函数的
- **HTTPException 对应的处理函数**
如果状态码是 204 或 304 返回 `Response(status_code=exc.status_code, headers=exc.headers)` 否则返回 `PlainTextResponse(exc.detail, status_code=exc.status_code, headers=exc.headers)`
- **WebSocketException 对应的处理函数**
直接关闭 socket 连接

**异常处理的过程**

 - 如果异常类型是 `HTTPException`, 那么会检查该异常对应状态码是否有对应的处理函数
 - 如果没有 Http 异常对应的处理函数, 就根据 异常类型 再次检查该异常类型是否有对应的处理函数.
 - 如果都没有找到异常处理函数就会抛出异常. 该异常会由上一层的 `ServerErrorMiddleware` 捕获处理
**Tips:** 
		

> 请求处理过程中,如果想要直接返回HTTP 错误, 可以在代码中抛出HTTPExcption 异常, 但是要注意,一般HTTPException 的状态码应该是 400 以上

### 1.1.4. Router

Router 有两个作用

 - 匹配客户端请求的路径找对对应的业务处理函数
 - asgi `startup` 和 `shutdown` 的执行
 

**1、startup 和 shutdown 部分**

```python
async def __call__(self, scope: Scope, receive: Receive,
                   send: Send) -> None:
    """
    The main entry point to the Router class.
    """
    assert scope["type"] in ("http", "websocket", "lifespan")

    if "router" not in scope:
        scope["router"] = self

    if scope["type"] == "lifespan":
        await self.lifespan(scope, receive, send)
        return
    # ...


async def lifespan(self, scope: Scope, receive: Receive,
                   send: Send) -> None:
    """
    Handle ASGI lifespan messages, which allows us to manage application
    startup and shutdown events.
    """
    started = False
    app = scope.get("app")
    await receive()
    try:
        async with self.lifespan_context(app):
            await send({"type": "lifespan.startup.complete"})
            started = True
            await receive()
    except BaseException:
        exc_text = traceback.format_exc()
        if started:
            await send({
                "type": "lifespan.shutdown.failed",
                "message": exc_text
            })
        else:
            await send({
                "type": "lifespan.startup.failed",
                "message": exc_text
            })
        raise
    else:
        await send({"type": "lifespan.shutdown.complete"})
```
如果实例化 Starlette 时传递的没有传递 lifespan ,那么会使用默认的 `lifespan`, 也就是使用调用 Router 对象的 startup 和 shutdown 执行存储的 `on_startup` 列表中的函数 和 `on_shutdown` 列表中的函数

```python
async def startup(self) -> None:
        """
        Run any `.on_startup` event handlers.
        """
        for handler in self.on_startup:
            if is_async_callable(handler):
                await handler()
            else:
                handler()

async def shutdown(self) -> None:
        """
        Run any `.on_shutdown` event handlers.
        """
        for handler in self.on_shutdown:
            if is_async_callable(handler):
                await handler()
            else:
                handler()
```
**2、路由匹配和执行**

```python
async def __call__(self, scope: Scope, receive: Receive,
                    send: Send) -> None:
     # ....

     partial = None

     for route in self.routes:
         # Determine if any route matches the incoming scope,
         # and hand over to the matching route if found.
         match, child_scope = route.matches(scope)
         if match == Match.FULL:
             scope.update(child_scope)
             await route.handle(scope, receive, send)
             return
         elif match == Match.PARTIAL and partial is None:
             partial = route
             partial_scope = child_scope

     if partial is not None:
         #  Handle partial matches. These are cases where an endpoint is
         # able to handle the request, but is not a preferred option.
         # We use this in particular to deal with "405 Method Not Allowed".
         scope.update(partial_scope)
         await partial.handle(scope, receive, send)
         return

     if scope["type"] == "http" and self.redirect_slashes and scope[
             "path"] != "/":
         redirect_scope = dict(scope)
         if scope["path"].endswith("/"):
             redirect_scope["path"] = redirect_scope["path"].rstrip("/")
         else:
             redirect_scope["path"] = redirect_scope["path"] + "/"

         for route in self.routes:
             match, child_scope = route.matches(redirect_scope)
             if match != Match.NONE:
                 redirect_url = URL(scope=redirect_scope)
                 response = RedirectResponse(url=str(redirect_url))
                 await response(scope, receive, send)
                 return

     await self.default(scope, receive, send)
```

```python
partial = None

for route in self.routes:
    # Determine if any route matches the incoming scope,
    # and hand over to the matching route if found.
    match, child_scope = route.matches(scope)
    if match == Match.FULL:
        scope.update(child_scope)
        await route.handle(scope, receive, send)
        return
    elif match == Match.PARTIAL and partial is None:
        partial = route
        partial_scope = child_scope
```
这里就是路由匹配的核心代码了, 遍历 Router 中的元素, 调用其 `matches` 方法进行匹配, 如果完全匹配了就调用该元素的 `handle` 方法, 传递(scope, receive, send) 给他, 然后就结束调用流程了
**Tips:** 

这里可以看出, 只要定义的 Route 对象具有 matches 方法 和 handle 方法就可以完成最基本路由匹配与执行, 所以完全可以自定义 Route 类型, 比如自定义一些嵌套的路由, 实际上 starlette 的 Mount 就是 `Route 的变体`

接下来看一下 Route 对象的`mathces` 和 `handle` 方法
**1、matches**

```python
def matches(self, scope: Scope) -> typing.Tuple[Match, Scope]:
    if scope["type"] == "http":
        match = self.path_regex.match(scope["path"])
        if match:
            matched_params = match.groupdict()
            for key, value in matched_params.items():
                matched_params[key] = self.param_convertors[key].convert(
                    value)
            path_params = dict(scope.get("path_params", {}))
            path_params.update(matched_params)
            child_scope = {
                "endpoint": self.endpoint,
                "path_params": path_params
            }
            if self.methods and scope["method"] not in self.methods:
                return Match.PARTIAL, child_scope
            else:
                return Match.FULL, child_scope
    return Match.NONE, {}
```
`self.path_regex.match(scope["path"])` 这一行是通过正则表达去匹配当前请求的路径, 这个正则表达式是在Route对象创建时构建的, 代码如下:

```python
CONVERTOR_TYPES = {
    "str": StringConvertor(),
    "path": PathConvertor(),
    "int": IntegerConvertor(),
    "float": FloatConvertor(),
    "uuid": UUIDConvertor(),
}
# ...
PARAM_REGEX = re.compile("{([a-zA-Z_][a-zA-Z0-9_]*)(:[a-zA-Z_][a-zA-Z0-9_]*)?}")
# ...
for match in PARAM_REGEX.finditer(path):
        param_name, convertor_type = match.groups("str")
        convertor_type = convertor_type.lstrip(":")
        assert (
            convertor_type
            in CONVERTOR_TYPES), f"Unknown path convertor '{convertor_type}'"
        convertor = CONVERTOR_TYPES[convertor_type]
        path_regex += re.escape(path[idx:match.start()])
        path_regex += f"(?P<{param_name}>{convertor.regex})"
# ...
```
如果匹配到了, 那么检查是否有路径参数, 如果有路径参数, 那么提取出路径参数, 并且对路径参数进行格式转换 `self.param_convertors[key].convert(value)`
**Tips:** 
路由参数进行格式转换是可以自定义的, 只要参考 `starlette.converters` 下的 `Converter` 类, 并且将其添加到 `CONVERTOR_TYPES` 中既可以

**2、handle**

```python
async def handle(self, scope: Scope, receive: Receive, send: Send) -> None:
    if self.methods and scope["method"] not in self.methods:
        headers = {"Allow": ", ".join(self.methods)}
        if "app" in scope:
            raise HTTPException(status_code=405, headers=headers)
        else:
            response = PlainTextResponse("Method Not Allowed",
                                         status_code=405,
                                         headers=headers)
        await response(scope, receive, send)
    else:
        await self.app(scope, receive, send)
```
这里先判断请求方法是否在方法列表中, 如果不再直接抛出 **405 Http** 异常

否则调用 `Route.app(scope,receive,send)` 那么这里关键的代码就是 app 这个属性中了, 看一下属性的内容

```python
def request_response(func: typing.Callable) -> ASGIApp:
    """
    Takes a function or coroutine `func(request) -> response`,
    and returns an ASGI application.
    """
    is_coroutine = is_async_callable(func)

    async def app(scope: Scope, receive: Receive, send: Send) -> None:
        request = Request(scope, receive=receive, send=send)
        if is_coroutine:
            response = await func(request)
        else:
            response = await run_in_threadpool(func, request)
        await response(scope, receive, send)

    return app

# ....

def __init__(
    # ...
    endpoint: typing.Callable,
    # ...
) -> None:
    # ...
    self.endpoint = endpoint
    # ...
    if inspect.isfunction(endpoint_handler) or inspect.ismethod(
            endpoint_handler):
        # Endpoint is function or method. Treat it as `func(request) -> response`.
        self.app = request_response(endpoint)
        if methods is None:
            methods = ["GET"]
    else:
        # Endpoint is a class. Treat it as ASGI.
        self.app = endpoint
    # ...
```
如果 `endpoint(也就是我们的视图函数)` 是函数对着方法, 那么就通过 `request_response` 包装一层, 执行的时候就是:

 - 如果是携程对象, 就直接 await
 - 如果是普通函数, 就再线程池中执行

**Tips:** 如果不是函数或者方法, 就把它当做一个普通类, 从代码中看出, 这个`endpoint` 类的实例化时应当接受(scope,receive,send)

```python
class Endp:

    async def __new__(cls, scope, receive, send, *args, **kwargs):
        pass


class Endpx:

    def __init__(self, scope, receive, send):
        pass

    def __await__(self):
        return 可等待对象
```

