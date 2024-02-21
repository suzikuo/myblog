---
title: app抓包
tags: 
- python
- spider
categories:
- python
date: 2021-12-24 15:59:21
---

- [一、fiddler配置](#一fiddler配置)
- [二、手机配置](#二手机配置)
- [三、Tunnel to](#三tunnel-to)


## 一、fiddler配置
![](https://img-blog.csdnimg.cn/8bcf34a0b32040948b42104eb8f4e430.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
![](https://img-blog.csdnimg.cn/6c59e2dabf1645468a1afe4d77aade9f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
![](https://img-blog.csdnimg.cn/042f1c9026b44d0db9cc5c65b7e1865f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
![](https://img-blog.csdnimg.cn/da9a0fffb60d4748a29315e80bc50687.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)

## 二、手机配置
1、设置wifi，
![](https://img-blog.csdnimg.cn/f47d29b4fba04debba0a5560886ec33d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
2、浏览器访问 ip:port  (172.168.3.243:8888)
	下载证书
![](https://img-blog.csdnimg.cn/a9361df74a904e7ead4e292c7d40b364.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
3、点击证书安装

## 三、Tunnel to

安装 xposed + justtrustme,当然安装这个的前提需要你的手机有 root 才行，如果没有话，推荐你使用 virtualXposed + justtrustme，这两个也行。

这里见到那说下 virtualXposed 的用法。

 

下载地址：https://github.com/android-hacker/VirtualXposed

插件下载：https://github.com/Fuzion24/JustTrustMe
![](https://img-blog.csdnimg.cn/1186af7886a14453995fb80a66c54dad.png)
![](https://img-blog.csdnimg.cn/9009516da13f4976a923217f7c8a730e.png)
![](https://img-blog.csdnimg.cn/0dff397b9b00455685fbd101658d39ef.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_14,color_FFFFFF,t_70,g_se,x_16)
![](https://img-blog.csdnimg.cn/afab1c24779e43048b6601b3a3b73372.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_17,color_FFFFFF,t_70,g_se,x_16)
![](https://img-blog.csdnimg.cn/6b3f53571523407a902d6840a36bff7a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_17,color_FFFFFF,t_70,g_se,x_16)
![](https://img-blog.csdnimg.cn/82f693c5ac2a4c319ba321671618d757.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_20,color_FFFFFF,t_70,g_se,x_16)
![](https://img-blog.csdnimg.cn/60605679d4bd452c84a0c97d6914036e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA6buR54yr4LmR,size_18,color_FFFFFF,t_70,g_se,x_16)
再在 virtualXposed 上面打开小红书软件，再看 fiddler，包都有了。

ps: 不支持32位应用时，使用0.18.2版本
[下载链接，密码vvgd](https://pan.baidu.com/s/1uQpIffFTFmqfsaq1knEqnA)
