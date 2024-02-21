---
title: python firda hook配置
tags: 
- python
- spider
categories:
- python
date: 2021-12-24 11:35:37
---

- [一、firda 安装](#一firda-安装)
- [二、安装木木模拟器](#二安装木木模拟器)
- [下载frida服务端-Android](#下载frida服务端-android)
- [使用adb连接mumu模拟器](#使用adb连接mumu模拟器)
- [python调试](#python调试)
- [脱壳](#脱壳)
- [查看](#查看)

## 一、firda 安装
	

```python
pip install firda -i http://pypi.douban.com/simple/ 
pip install frida-tools -i http://pypi.douban.com/simple/ 
```

## 二、安装木木模拟器
[官网： http://mumu.163.com/](http://mumu.163.com/)
**开启root**
![](https://img-blog.csdnimg.cn/965faa30d7654b16b16084cbc4b9322d.png)
![](https://img-blog.csdnimg.cn/1a557606f2d547db8ad8cd11ae18f9c3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
**打开模拟器USB调试
在开发者选项中。**
![](https://img-blog.csdnimg.cn/19503c773a9e4452a7d7b5e6551f1c7f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
**安装RE文件管理器
直接搜索进行下载,下载后打开给予root权限**
![](https://img-blog.csdnimg.cn/6d104811791a40f1ba18bdde5ec19aea.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)

## 下载frida服务端-Android

**下载frida-server文件**
[到 https://github.com/frida/frida/releases 下载相应的版本（版本一定对应firda版本）](https://github.com/frida/frida/releases)
**移动文件到tmp下**
下载完成之后,如果你使用的本地下载,则把这个文件放入木木共享文件夹中解压,重命名为 frida-server

**![](https://img-blog.csdnimg.cn/0e068243f7dc4b94aa483c11223cebfd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
**
然后打开 RE文件管理器,使用全局搜索,找到这个文件的位置.
![](https://img-blog.csdnimg.cn/7078fa0d8c034015b578ede0bf8de7b3.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
接着把 frida-server文件复制到 /data/local/tmp 目录下.
![](https://img-blog.csdnimg.cn/acc533f0aa5a4afca3f34fb083a6fe3d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)

## 使用adb连接mumu模拟器
先找到模拟器的安装目录,然后进入 emulator\nemu\vmonitor\bin目录
![](https://img-blog.csdnimg.cn/dbba4d33a4b447c28971571a0ac861c4.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)

配置adb
将adb_server.exe 重命名为adb.exe
配置全局变量
![](https://img-blog.csdnimg.cn/996389ad4b8a47adbe8a6fd220ccfdb8.png)
1、打开cmd，输入 adb connect 127.0.0.1:7555
2、adb shell 进入交互环境
3、cd  /data/local/tmp
4、chmod 777 frida-server  设置权限
5、./frida-server   启动服务
6、adb forward tcp:27042 tcp:27042    开放访问端口
7、打开新cmd，输入frida-ps -U，有信息则服务启动成功

## python调试

```python
import frida
import sys

#获取设备信息
rdev = frida.get_remote_device()

#获取在前台运行的APP
front_app = rdev.get_frontmost_application()
print (front_app)

```
**案例**
```python
import frida
rdev = frida.get_remote_device()
#获取在前台运行的APP
front_app = rdev.get_frontmost_application()
print (front_app)
import re
res = re.findall(r"pid=(.*?),",str(front_app))
# 枚举进程中加载指定模块中的导出函数
session = rdev.attach(int(res[0]))   # 也可以使用attach(pid)的方式
jscode = """
Java.perform(function(){
    var temp = Java.use('com.xunyou.libservice.server.impl.ServerApi');
    var ServerParams = Java.use('com.xunyou.libservice.server.impl.ServerParams');
        temp.e.overload('org.json.JSONObject').implementation = function(a){
            var result = this.e(a);  
            send(hasmap_parse(result));
            return result;
            }

    function hasmap_parse(result){
     var test = result.toString();
            var keyset = result.keySet();
            var it = keyset.iterator();
            while(it.hasNext()){
                var keystr = it.next().toString();
                var valuestr = result.get(keystr).toString();
                test +=keystr+":"+ valuestr+";";
            }
        
        return test;
    }
});
"""

script = session.create_script(jscode)
def on_message(message, data):
    print(message)
script.on('message', on_message)
script.load()
sys.stdin.read()
```

## 脱壳
使用frida_dexdump 强制脱壳，可脱壳已加固
[下载地址](https://github.com/hluwa/FRIDA-DEXDump)

**使用**
在模拟器上运行app后，执行 python main.py
脱壳后的文件将保存到当前目录
![](https://img-blog.csdnimg.cn/0b72dbbf35f946ac87c728e2ec754900.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_14,color_FFFFFF,t_70,g_se,x_16)

## 查看
使用jadx-gui查看源代码（拖进去即可）
![](https://img-blog.csdnimg.cn/f0e7f3b82ecf471c881ea461e28c4265.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)

