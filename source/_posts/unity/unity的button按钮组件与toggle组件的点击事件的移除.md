---
title: unity的button按钮组件与toggle组件的点击事件的移除
tags: 
- unity
categories:
- unity
date: 2021-08-10 16:21:16
---

button按钮添加点击事件可以使用这种形式：
![](https://img-blog.csdnimg.cn/63db9678898a478599560035ef6a3009.png)
移除某个事件使用SonObj.GetComponent<Button>().onClick.RemoveListener（call）方法移除某个事件。
RemoveListener中需要传递的参数为 UnityAction类型：
![](https://img-blog.csdnimg.cn/af95d67fa5644bf68d03710535017543.png)
因此，在创建监听事件时，可以将点击事件提取出来，
![](https://img-blog.csdnimg.cn/e853bbcdfba04de1b11da2f98904e307.png)
在想要移除此次添加的点击事件时，将call传递给RemoveListener就行。
![](https://img-blog.csdnimg.cn/a33d2fd5567f4b758d95cd98f0e2dabd.png)

