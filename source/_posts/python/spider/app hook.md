---
title: app hook
tags: 
- python
- spider
categories:
- python
date: 2021-12-27 15:36:56
---

- [一、使用jadx查看代码](#一使用jadx查看代码)
- [二、apktool](#二apktool)
- [代码](#代码)


案例：
案例apk：[下载地址，提取码4274](https://pan.baidu.com/s/1chb-sFbrXo8h0hwMIxd3cw)

## 一、使用jadx查看代码

app界面(猜拳小游戏)：
![](https://img-blog.csdnimg.cn/99af538b520146c48fadbe523166d2c9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
打开 androidmanifest.xml 
![](https://img-blog.csdnimg.cn/baff00dc607c4802b45e5a362228b3ff.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
查找activity，此为实例与组件的对应。因为案例只有一个界面，所以目前只有一个activity。
根据android:name 找到对应类
![](https://img-blog.csdnimg.cn/497b76d763134c4582e2fedb5edfbe7f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
![](https://img-blog.csdnimg.cn/8d64a51abf194ee2bff2726a954ea6ac.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)

在代码中可以看到showMessageTask 为一个类的实例，因此想要hook此类，需要使用apktool反编译

## 二、apktool
下载apktool后，打开cmd，进入此下载目录。
输入命令`apktool.bat d F:\Desktop\books爬虫\lesson2-rps-frida-test.apk`
![](https://img-blog.csdnimg.cn/77c6c67c56c543fab2aca27a07f7eea1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
此时，会在当前目录下生成和apk名字相同的文件夹，进入`lesson2-rps-frida-test\smali\com\example\seccon2015\rock_paper_scissors`
可以看到smail文件
![](https://img-blog.csdnimg.cn/ab3eea8deb854f2db1982be2bbe36a56.png)
**MainActivity.smali**文件为我们之前找到的类
**MainActivity$1.smali**就是我们需要hook的属性为类类型的类了。
hook时，使用**包名+类名**即可hook
## 代码

```python
import codecs
import frida
import sys

hook_code = '''
Java.perform(
    function () {
        console.log('[*] Running CTF')
        var MainActivity = Java.use('com.example.seccon2015.rock_paper_scissors.MainActivity')
        MainActivity.onClick.implementation = function(v){
            // m 是默认值
            //console.log('m is :'+this.m.value)
            //console.log('n is :'+this.n.value)
            this.onClick(v)
            console.log('m is :'+this.m.value)
            console.log('n is :'+this.n.value)
        }
        var TT = Java.use('com.example.seccon2015.rock_paper_scissors.MainActivity$1') // 从 Smali 文件找
        TT.run.implementation = function(){
            // 内部类访问外部类的成员变量
            // this.this$0.value.外部类属性名.value 
            // this.m.value = 1    // TypeError: cannot write property 'value' of undefined
            // this.n.value = 2
            this.this$0.value.m.value = 1
            this.this$0.value.n.value = 2
            this.run()
        }  
    }
)
'''


def on_message(message, data):
    if message['type'] == 'send':
        print(message['payload'])
    elif message['type'] == 'error':
        print(message['stack'])


# hook 已经启动的app
process = frida.get_usb_device(timeout=1000).attach('com.example.seccon2015.rock_paper_scissors')
script = process.create_script(hook_code)
script.on('message', on_message)
script.load()
sys.stdin.read()  # 防止程序退出

# spawn 重启app 可以hook app的启动阶段 (可能会使app闪崩)
# device = frida.get_usb_device(timeout=1000)
# pid = device.spawn(['com.example.seccon2015.rock_paper_scissors'])
# process = device.attach(pid)
# script = process.create_script(hook_code)
# script.on('message', on_message)
# script.load()
# sys.stdin.read()  # 防止程序退出

```

