---
title: python-爬虫-woff静态字体-映射方法之一（大众点评字形与编码的对应关系）
tags: 
- python
- spider
categories:
- python
date: 2020-05-12 19:21:58
---

woff字体可在[font editor](http://fontstore.baidu.com/static/editor/index.html)查看映射关系

> 直接看这位大佬写的吧；woff文件解析https://blog.csdn.net/weixin_43752839/article/details/98314821#commentBox

思路：


	1、通过selenium截取整个页面
	
	2、使用Image截取每个字体，
	
	3、使用图片识别技术（推荐百度云文字识别）
	
```python
import os
from time import sleep

from PIL import Image
from selenium import webdriver
driver = webdriver.Chrome('D:\chromedriver_win32\chromedriver.exe')
driver.get('http://fontstore.baidu.com/static/editor/index.html')
#!!!!需手动上传文件！！
sleep(10)#上传文件没有写，留10秒时间上传文件，并设置网页格式，保证整个页面能够显示完整当前页面的字体

t = True		#退出关键词
while t:
	sleep(5)
	#1级文件夹
    path = 'woff/'
    #获取当前页码，保证正常存储
    page = driver.find_element_by_xpath('//input[@data-pager="text"]').get_attribute('value')
    print(page)
    if not os.path.exists(path+page):
    	#2级文件夹
        os.mkdir(path+page)
        
	#适当的调整节点大小，使文字能够完全显示出来。
    driver.execute_script('var l = document.getElementsByClassName("glyf");for(var i=0;i<l.length;i++){l[i].style.height ="100px"}')

	#屏幕截图
	driver.get_screenshot_as_file(path+page+'.png')
	#获取字体父元素节点列表
	font_list = driver.find_elements_by_class_name('glyf-item')
	#调整上一步截取的图片像素，保证图片像素与网页大小差不多
	pic = Image.open(path+page+'.png')
    pic = pic.resize((1536,749))
    pic.save(path+page+'.png')
    pic = Image.open(path+page+'.png')

	#一般woff字体中前两个为垃圾字符，可以去除
    if page ==1:
        font_list = font_list[2:]

	#遍历字体父元素节点列表
	 for font in font_list:
		#获取字体所在节点
        svg = font.find_element_by_class_name('glyf')
        #获取字体所对应的unicode编码
        uni = font.find_element_by_class_name('name').text or 'a'
        print(uni)
        #字体截图
        pic1 = pic.crop((svg.rect['x'], svg.rect['y'], svg.rect['x'] + svg.rect['width'], svg.rect['y'] + svg.rect['height']))
        print(path + page + '/' + uni + '.png')
        #保存图片
        pic1.save(path+page+'/'+uni + '.png')


	#下一页节点
	next_button = driver.find_element_by_xpath('//button[@data-pager="next"]')
    try:
    	#判断是否为最后一页，如果不是，则pass
        d = driver.find_element_by_xpath('//button[@data-pager="next"]/@data-disabled')

        t = False
    except:
        pass
    next_button.click()

```



	到此，全部的字体已经截图完成。接下来将进行文字识别。
	使用到的是调用百度文职识别api接口，
[https://ai.baidu.com](https://ai.baidu.com)
	登录百度账号，进去控制台，选择文字识别，创建应用，并复制三个参数
	![](https://img-blog.csdnimg.cn/20200512190136521.png)
	

```python
#picfile:图片路径
#name：图片名字
#n：用来保证容错率

def baiduOCR(picfile,name,n):
    # filename = path.basename(picfile)
    APP_ID = '19845704'
    API_KEY = 'u9fCGvIPf4P1FbtLOQDWebRS'
    SECRECT_KEY = 'kNo3Hrz0ucoaqFrtBYNT2PjHDrHbcK3u'
    client = AipOcr(APP_ID, API_KEY, SECRECT_KEY)
    i = open(picfile, 'rb')
    img = i.read()

    message = client.basicGeneral(img) #普通识别
    #message = client.basicAccurate(img)#高级识别
    i.close()
   
	n+=1

	try:
        for text in message.get('words_result'):
        	#确认是否识别出文字内容
            if len(text.get('words')) > 1:
                if n < 3:
                    baiduOCR(picfile, name, n)
                else:
                    return False
            else:
                return text.get('words')
    except Exception as e:
        pass
    if n < 3:
        baiduOCR(picfile, name, n)
    else:
        return False



def ab(picfile,name,n):
	#failure.txt:用于存放普通识别失败的图片地址
	#success.txt：用于存放识别成功的数据
    s = baiduOCR(picfile, name, n)
    print(s)
    d = True
    if s == False or s == None:
        print('识别失败：' + picfile)
        with open('failure.txt', 'a+') as f:
            f.write(picfile+'\n')
        d = False
    if d:
    	#此处是我为了方便写的，
        t = '"' + name.split('.')[0] + '"' + ':' + '"' + s + '"'
        print(t)
        with open('succcess.txt','a+') as f:
            f.write(t+'\n')


if __name__ == '__main__':
	
    path = os.listdir('woff')
    dir_list = []
    a = {}
    n = 0
    for p in path:
        if os.path.isdir('woff/'+p):
            dir_list.append('woff/'+p)
    for l in dir_list:
        print(l)
        png = os.listdir(l)
        for p in png:
            ab(l+'/'+p,p,n)



```
**最后，再使用高级识别接口（client.basicAccurate(img），进行第二次识别，解决上一步识别失败的图片，剩余的再手动映射吧。**



