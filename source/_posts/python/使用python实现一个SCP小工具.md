---
title: 使用python实现一个SCP小工具
tags: 
- python
categories:
- python
date: 2024-2-26 11:23:11
---

**源码地址：**
[ssh_scp](https://github.com/suzikuo/ssh_scp)
**工具截图：**

> 一个简易的scp文件上传下载小工具，用来上传或下载一些小文件。
> 目前只适用于windows，


![在这里插入图片描述](https://img-blog.csdnimg.cn/direct/a599a39ab11d418daf5cad9dabc27264.png)

## **使用方法：**

### 前提：
工具同级目录，创建一个`ssh_commands.json`文件。用来存储配置信息。

```bash
{
    "Show Name ": { # 显示的名称
        "pem": "C:\\Users\\xxx\\Desktop\\pem\\VPN.pem", # PEM文件地址
        "host": "ubuntu@ec2-xxxxxx.ap-northeast-2.compute.amazonaws.com" # 机器IP地址
    },
    "Show Name2 ": { # 显示的名称
        "password": "123123123", # 密码
        "host": "root@192.168.1.1" # 机器IP地址
    }
}
```

### **上传：**
1、选择要上传的文件或文件夹。
2、在SSH连接中，选择目标机器，点击选择连接。
3、在远程文件中，输入目标机器的存储地址
4、点击上传。
### 下载：
1、选择要下载到的文件夹地址，例如桌面。
2、在SSH连接中，选择源机器，点击选择连接。
3、在远程文件中，输入要下载的文件的地址。
4、点击下载。
