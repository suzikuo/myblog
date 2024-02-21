---
title: flask migrate创建迁移脚本无效
tags: 
- python
- flask
categories:
- python
date: 2020-04-07 17:28:25
---
前面的步骤就先不说了，也就是初始化migrate、绑定Manage。
1、首先使用init初始化


![](https://img-blog.csdnimg.cn/20200407172128417.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dtamYx,size_16,color_FFFFFF,t_70)
2、生成迁移文件

![](https://img-blog.csdnimg.cn/20200407172335172.png)
此时可能会出现如图问题，前提是已经创建model。
问题所在：创建的model对象系统不知道，没有告诉系统，也就是没有实例化对象。因此需要在views中导入要迁移的model，系统才知道要迁移哪个模板。

views中导入相应模块
![](https://img-blog.csdnimg.cn/20200407181334290.png)


