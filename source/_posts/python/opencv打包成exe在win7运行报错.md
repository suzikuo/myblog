---
title: opencv打包成exe在win7运行报错
tags: 
- python
categories:
- python
date: 2021-03-26 11:26:33
---

## opencv打包成exe在win7运行报错

**1、安装过程错误警告**
opencv版本：3.4.3
打包环境：win10
运行环境：win7
opencv报错：

![](https://img-blog.csdnimg.cn/20210326111826691.png)

**2、解决方案**
[opencv插件打包](https://pan.baidu.com/s/1qXt1BNClftx6fE5aAQWrsw)
提取码: fp7v 
将 api-ms-win-downlevel-shlwapi-l1-1-0.dll【System32】 放到 c:\windows\System32
将api-ms-win-downlevel-shlwapi-l1-1-0.dll【SysWOW64】 放到C:\Windows\SysWOW64
打包环境与运行环境都需要添加dll文件；重新打包、安装；
**3、注意**
opencv>3.4.3将出现:
	CAP_IMAGES: can‘t find starting number (in the name of file)
	错误，提示缺少文件，或没有编号。暂无解决。
