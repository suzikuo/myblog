---
title: python中threading开启关闭线程
tags: 
- python

categories:
- python
date: 2021-03-18 20:04:25
---

## python中threading开启关闭线程
> https://blog.csdn.net/qq_15181569/article/details/93299164

[python中threading开启关闭线程](https://blog.csdn.net/qq_15181569/article/details/93299164)


**一、启动线程**
首先导入threading

```python
myThread = threading.Thread(target=serial_read)
myThread.start()
```
**二、停止线程**
不多说了直接上代码

```python
import inspect
import ctypes
def _async_raise(tid, exctype):
    """raises the exception, performs cleanup if needed"""
    tid = ctypes.c_long(tid)
    if not inspect.isclass(exctype):
        exctype = type(exctype)
    res = ctypes.pythonapi.PyThreadState_SetAsyncExc(tid, ctypes.py_object(exctype))
    if res == 0:
        raise ValueError("invalid thread id")
    elif res != 1:
        # """if it returns a number greater than one, you're in trouble,
        # and you should call it again with exc=NULL to revert the effect"""
        ctypes.pythonapi.PyThreadState_SetAsyncExc(tid, None)
        raise SystemError("PyThreadState_SetAsyncExc failed")
def stop_thread(thread):
    _async_raise(thread.ident, SystemExit)
```

```python
stop_thread(myThread)
```

