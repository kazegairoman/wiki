## 首先要找到所有文章链接地址

一般来说，网上是搜不到某个公众号的全部文章的。但是我们利用 **微信公众号平台** 就能够一次性找到。

要用到它，肯定需要先注册登录一个微信公众号。当我们来到后台界面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/1302c4024b064903b2287edc673cd2e9.png)

进入**素材管理**——> **新建图文**——> **（上方的）超链接**界面

![在这里插入图片描述](https://img-blog.csdnimg.cn/8e73f16450764257b87089e1e5395dfc.png)  
**当我们打开开发者工具，搜索点击某个公众号后，就会抓到一个get请求包**  
![在这里插入图片描述](https://img-blog.csdnimg.cn/5a02dd3e080c4519acb70eae81ffcb9f.png)  
**它返回的就是文章相关的信息**（包含了，文章链接，文章标题，创建时间，封面地址等）  
![在这里插入图片描述](https://img-blog.csdnimg.cn/783fe7b39a474c238d01f567ac6cce97.png)

### 因此找到请求后，我们开始构造

url 肯定是不变的 https://mp.weixin.qq.com/cgi-bin/appmsg  
请求头(request headers)复制粘贴过去  
(在代码中只需要复制cookie和user-agent)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7e4f1cee35384802b492b9ecac870954.png)  
get请求的参数params也能直接进行复制  
![在这里插入图片描述](https://img-blog.csdnimg.cn/8e617cf124174cdbb5a04c44066064c2.png)  
**这里的begin代表的是页数，如果需要更多数据，肯定是需要翻页的。多次请求改变begin的值就可以了**

第一部分的代码如下

```python
import requests
import time
import datetime
import pytz

url="https://mp.weixin.qq.com/cgi-bin/appmsg"

#我这里先把数据写入到txt文件中

f=open("微信公众号文章.txt",mode="w",encoding="utf-8")
headers={
		#这里是你自己的headers
		#需要cookie和user-agent
}
params={
		#这里是你自己的params
		#全部复制
}
begin=0

#发起第一次请求
resp=requests.get(url,headers=headers,params=params,verify=False)

data = resp.json()
while True:
    time.sleep(5)
    
    
    #data['app_msg_list']里面装载的就是我们需要的数据
    if len(data['app_msg_list'])==0:
    	#可能数据请求完了，可能请求失败，可能你的微信公众号请求被禁用一段时间（因为爬数据过于频繁，被发现了）
        break
    for app_msg in data['app_msg_list']:
        title = app_msg['title']  #文章标题
        link = app_msg['link']    #文章链接
        update_time=app_msg['update_time']   #文章创建时间，为时间戳
        #将时间戳转为 年月日，时分秒
        update_time = datetime.datetime.fromtimestamp(int(update_time), pytz.timezone('Asia/Shanghai')).strftime('%m/%d/%Y %H:%M:%S')  
        cover =app_msg['cover']  #文章封面图片地址
    
    #请求下一页的数据
    begin+=1
    params['begin']=begin*5

    resp = requests.get(url, headers=headers, params=params,verify=False)
    data = resp.json()
print(resp)
```

****要值得注意的是，在第一步时候，如果请求过于频繁，Cookie会被服务器暂时封禁（自测是几十分钟）。每次请求之前间隔几秒休息，也不太好使。这时候就只能换一个公众号或者等个几十分钟。再次运行。（一次能抓60页左右的数据）****

## python爬取html文件

使用python爬取某网站首页并下载html文件

下面介绍两种方式，一种是**urllib**，另一种是**requests**

**1、使用urllib**

```python

import urllib.request
url = 'http://www.baidu.com/'

# 向指定的url发送请求，并返回服务器响应的类文件对象
request = urllib.request.Request(url)

# 类文件对象支持 文件对象的操作方法，如read()方法读取文件全部内容，返回字符串
response = urllib.request.urlopen(request)
html = response.read()

with open('baidu.html', 'wb') as f:   
    f.write(html)

# 打印字符串
print('ok')

```

**2、使用requests**

```python

import requests

url = 'https://www.baidu.com'

response = requests.get(url) 

string = (response.text).encode()

with open('baidus.html', 'wb') as f:
    f.write(string)

# 打印字符串
print('ok')
```

## 完整代码

```python
import urllib.request

import requests

import time

import datetime

import pytz

import requests

from bs4 import BeautifulSoup

import os

import re

import scrapy

from urllib.parse import urlparse

  
  

url="https://mp.weixin.qq.com/cgi-bin/appmsg"

  

#我这里先把数据写入到txt文件中

  
f = open("微信公众号文章.txt",mode="w",encoding="utf-8")

headers={

        #这里是你自己的headers

        "Cookie": '***',

        "User-Agent": 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/115.0.0.0 Safari/537.36 Edg/115.0.1901.188',

}

begin = 0

params={

        #这里是你自己的params

        "action": "list_ex",

        "begin": begin,

        "count": "5",

        "fakeid": "Mzg3Mzg3NDg1Mg==",

        "type": "9",

        "token": '690725819',

        "lang": "zh_CN",

        "f": "json",

        "ajax": "1"

}

  
  

#发起第一次请求

resp=requests.get(url,headers=headers,params=params,verify=False)

  

data = resp.json()

while True:

    time.sleep(5)

    #data['app_msg_list']里面装载的就是我们需要的数据

    if len(data['app_msg_list'])==0:

        #可能数据请求完了，可能请求失败，可能你的微信公众号请求被禁用一段时间（因为爬数据过于频繁，被发现了）

        break

    for app_msg in data['app_msg_list']:

        title = app_msg['title']  #文章标题

        link = app_msg['link']    #文章链接

        update_time=app_msg['update_time']   #文章创建时间，为时间戳

        #将时间戳转为 年月日，时分秒

        update_time = datetime.datetime.fromtimestamp(int(update_time), pytz.timezone('Asia/Shanghai')).strftime('%m/%d/%Y %H:%M:%S')  

        cover =app_msg['cover']  #文章封面图片地址

        # 将数据写入txt文件

        f.write(f"标题：{title}\n链接：{link}\n创建时间：{update_time}\n封面链接：{cover}\n\n")

        # 以文章名为名称生成html文件

        response=urllib.request.urlopen(link)

        html=response.read().decode('utf-8')

        with open(f'html/{title}.html','w',encoding='utf-8') as d:

            d.write(html)

    #请求下一页的数据

    begin += 1

    params['begin']=begin*5

  

    resp = requests.get(url, headers=headers, params=params,verify=False)

    data = resp.json()

  

f.close()

print(resp)
```