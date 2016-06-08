---
title: Auto post
date: 2016-06-08 15:38:59
tags:
 - code
 - python
 - web
---

### 背景

某个坑爹的小伙伴说只要我做出这玩意就请我吃3斤龙虾，于是我本着社会主义精神，昧着虾心，来写这篇文章。

主要用的知识点：

1. python编程
2. http协议包分析
3. requests库文件的使用

预备环境：

1. windows平台
2. python2.7
3. requests库
4. python路径已经添加至环境变量，requests库已经安装

### 抓包分析

首先我们使用chrome 打开地址[http://comment.news.163.com/news_guonei8_bbs/BNCFDKN900014PRF.html](http://comment.news.163.com/news_guonei8_bbs/BNCFDKN900014PRF.html)

这是我要顶的那个贴的地址。

随意登录一个帐号（我们要跟帖，必须要登录帐号）

然后按`F12`打开开发者工具，并切换到`Network`选项卡。

{% asset_img  1.png %}

接着我们输入我们要跟帖的文字，注意，文字内容必须健康向上符合《互联网管理办法》，然后点击`马上发表`

{% asset_img  2.png %}

我们在第三个请求包中发现了我们传输过去的内容，同时发现我们的跟帖正在被审核（这边我不清楚是否是人工审核，讲道理应该不会是人工审核，人工审核的话，用自动化脚本来跟帖会比较的麻烦）

{% asset_img  3.png %}

点击`view source` 可以更详细的看到协议包的内容。

{% asset_img  4.png %}

我们可以看到，这是一个POST请求包，所以我们需要构造一个post的请求包，有一些字段是没有用的，这里只说对我们跟帖起到作用的字段：

```
POST /api/v1/products/a2869674571f77b5a0867c3d71db5856/threads/BNCFDKN900014PRF/comments?ibc=newspc HTTP/1.1
Host: comment.news.163.com

##这是贴子的地址
```

`cookie`这是用来验证用户身份的。
`Form data`这是urlencode过的数据，点击view source便可以看到，是我们发表的内容。

{% asset_img  5.png %}
接下来便是构造请求包了

### 请求包构造

我们这里使用的是python的requests的库。一个简单的请求如下
```
from requests import *
r = get("http://www.baidu.com")
print r.content

## 这样我们就对百度首页进行了一次get请求

```

我们也可以换成`post`

```
from requests import *
r = post("http://www.baidu.com")
print r.content

## 一次post请求便完成了
```

至此我们解决了请求方式的问题以及请求地址的问题。
我们还需要解决的是cookie的问题和form data的问题。
查看官方文档http://www.python-requests.org/en/master/user/quickstart/#cookies

{% asset_img  6.png %}

以下是我们请求包里面的cookie值
```
_ntes_nuid=910ac3d80a183cd3806dd61479b09d76; usertrack=c+5+hlb7Xhm3WHMlAxSMAg==; _ntes_nnid=b99c6cd761700dcd9345852c71c6137e,1459314268699; _ga=GA1.2.1744200486.1459314269; JSESSIONID-WNC-98XSE=91eb6ff75c89cf758a1a778ff385f3f3a74f1121d1c717f361c42438c4b861d1bee3860c17f7152d419286d103fcd9760e369e6177e38ab70a4d3896743946ce45cd2fd4b0cef4e9ab50fa9047eac2b40960b83b08ffd6ea6306208ac98d45208b87244e260fead013d6c9f3dd8078a8c5d046153453d4c0048246af85b65f19ed6d791d%3A1463715848426; _lkiox7665q_=26; NTES_SESS=UwvXj.cllvWvGy06Xm5xYdTV65rofunn.uAvp7LJVuC3krW0kCD17oXOuWkN13rSNfN24o9GOSeZ_RzjzJBK52oIUbeglGU9r6FIYdH33u4KPMqU1PAMgi8FaUMyCHbHJPZHabBJF3yw1OxP5WeKWrGwMgjUrlVzi95aEMdWPPSsHdX.mY1BQdlxAPegQ0tDS; S_INFO=1463712243|0|2&70##|m18258122031_1; P_INFO=m18258122031_1@163.com|1463712243|1|content|00&99|null&null&null#zhj&330400#10#0#0|182031&1||18258122031@163.com; NTES_PASSPORT=1t7E4WQRb3Gqgim1fjBWGBw22D8bHgaP.vGnIggM7YVj7LHE7Muc9bwRsH7TclLKTywQ6tmMF8hT7Z6XsXmqM5WRpL2ENY3u4pyXoQBxiRITx; ANTICSRF=b52fe9b761cd7166754a07a2814a3956; NTES_CMT_USER_INFO=99679345%7Cm182****2031_1%7C%7Cfalse%7CbTE4MjU4MTIyMDMxXzFAMTYzLmNvbQ%3D%3D; n_ht_s=1; cm_newmsg=user%3Dm18258122031_1%26new%3D-1%26total%3D-1; NTES_REPLY_NICKNAME=m18258122031_1%40163.com%7Cm18258122031_1%7C%7C%7C%7C1t7E4WQRb3Gqgim1fjBWGBw22D8bHgaP.vGnIggM7YVj7LHE7Muc9bwRsH7TclLKTywQ6tmMF8hT7Z6XsXmqM5WRpL2ENY3u4pyXoQBxiRITx%7C1%7C-1; cmt_vin=e754154fe4a567dd44864ffe1658fa37e24258408af0e2e319bbe3bbccdfb0f2
```
然后我们按照官方文档所说的，构造cookie字段。
```
cookies = {'_ntes_nuid':'910ac3d80a183cd3806dd61479b09d76','usertrack':'c+5+hlb7Xhm3WHMlAxSMAg==','_ntes_nnid':'b99c6cd761700dcd9345852c71c6137e,1459314268699','_ga':'GA1.2.1744200486.1459314269','JSESSIONID-WNC-98XSE':'91eb6ff75c89cf758a1a778ff385f3f3a74f1121d1c717f361c42438c4b861d1bee3860c17f7152d419286d103fcd9760e369e6177e38ab70a4d3896743946ce45cd2fd4b0cef4e9ab50fa9047eac2b40960b83b08ffd6ea6306208ac98d45208b87244e260fead013d6c9f3dd8078a8c5d046153453d4c0048246af85b65f19ed6d791d%3A1463715848426','_lkiox7665q_':'26','NTES_SESS':'UwvXj.cllvWvGy06Xm5xYdTV65rofunn.uAvp7LJVuC3krW0kCD17oXOuWkN13rSNfN24o9GOSeZ_RzjzJBK52oIUbeglGU9r6FIYdH33u4KPMqU1PAMgi8FaUMyCHbHJPZHabBJF3yw1OxP5WeKWrGwMgjUrlVzi95aEMdWPPSsHdX.mY1BQdlxAPegQ0tDS','S_INFO':'1463712243|0|2&70##|m18258122031_1','P_INFO':'m18258122031_1@163.com|1463712243|1|content|00&99|null&null&null#zhj&330400#10#0#0|182031&1||18258122031@163.com','NTES_PASSPORT':'1t7E4WQRb3Gqgim1fjBWGBw22D8bHgaP.vGnIggM7YVj7LHE7Muc9bwRsH7TclLKTywQ6tmMF8hT7Z6XsXmqM5WRpL2ENY3u4pyXoQBxiRITx','ANTICSRF':'b52fe9b761cd7166754a07a2814a3956','NTES_CMT_USER_INFO':'99679345%7Cm182****2031_1%7C%7Cfalse%7CbTE4MjU4MTIyMDMxXzFAMTYzLmNvbQ%3D%3D','n_ht_s':'1','cm_newmsg':'user%3Dm18258122031_1%26new%3D-1%26total%3D-1','NTES_REPLY_NICKNAME':'m18258122031_1%40163.com%7Cm18258122031_1%7C%7C%7C%7C1t7E4WQRb3Gqgim1fjBWGBw22D8bHgaP.vGnIggM7YVj7LHE7Muc9bwRsH7TclLKTywQ6tmMF8hT7Z6XsXmqM5WRpL2ENY3u4pyXoQBxiRITx%7C1%7C-1','cmt_vin':'e754154fe4a567dd44864ffe1658fa37e24258408af0e2e319bbe3bbccdfb0f2'}
```
然后构造data发送post包
```
url  = 'http://comment.news.163.com/api/v1/products/a2869674571f77b5a0867c3d71db5856/threads/BNCFDKN900014PRF/comments?ibc=newspc'
data = dict(content='支持习总书记，捍卫祖国安全，相信能带领中国的百姓实现愿望。',parentId='',board='news_guonei8_bbs')
r = post(url, cookies=cookies,data=data)
```

整理下,脚本如下
```python
# -*- coding:utf8 -*-
from requests import *

url  = 'http://comment.news.163.com/api/v1/products/a2869674571f77b5a0867c3d71db5856/threads/BNCFDKN900014PRF/comments?ibc=newspc'

data = dict(content='支持习总书记，捍卫祖国安全，相信能带领中国的百姓实现愿望。',parentId='',board='news_guonei8_bbs')

cookies ={'_ntes_nuid':'910ac3d80a183cd3806dd61479b09d76','usertrack':'c+5+hlb7Xhm3WHMlAxSMAg==','_ntes_nnid':'b99c6cd761700dcd9345852c71c6137e,1459314268699','_ga':'GA1.2.1744200486.1459314269','JSESSIONID-WNC-98XSE':'91eb6ff75c89cf758a1a778ff385f3f3a74f1121d1c717f361c42438c4b861d1bee3860c17f7152d419286d103fcd9760e369e6177e38ab70a4d3896743946ce45cd2fd4b0cef4e9ab50fa9047eac2b40960b83b08ffd6ea6306208ac98d45208b87244e260fead013d6c9f3dd8078a8c5d046153453d4c0048246af85b65f19ed6d791d%3A1463715848426','_lkiox7665q_':'26','NTES_SESS':'UwvXj.cllvWvGy06Xm5xYdTV65rofunn.uAvp7LJVuC3krW0kCD17oXOuWkN13rSNfN24o9GOSeZ_RzjzJBK52oIUbeglGU9r6FIYdH33u4KPMqU1PAMgi8FaUMyCHbHJPZHabBJF3yw1OxP5WeKWrGwMgjUrlVzi95aEMdWPPSsHdX.mY1BQdlxAPegQ0tDS','S_INFO':'1463712243|0|2&70##|m18258122031_1','P_INFO':'m18258122031_1@163.com|1463712243|1|content|00&99|null&null&null#zhj&330400#10#0#0|182031&1||18258122031@163.com','NTES_PASSPORT':'1t7E4WQRb3Gqgim1fjBWGBw22D8bHgaP.vGnIggM7YVj7LHE7Muc9bwRsH7TclLKTywQ6tmMF8hT7Z6XsXmqM5WRpL2ENY3u4pyXoQBxiRITx','ANTICSRF':'b52fe9b761cd7166754a07a2814a3956','NTES_CMT_USER_INFO':'99679345%7Cm182****2031_1%7C%7Cfalse%7CbTE4MjU4MTIyMDMxXzFAMTYzLmNvbQ%3D%3D','n_ht_s':'1','cm_newmsg':'user%3Dm18258122031_1%26new%3D-1%26total%3D-1','NTES_REPLY_NICKNAME':'m18258122031_1%40163.com%7Cm18258122031_1%7C%7C%7C%7C1t7E4WQRb3Gqgim1fjBWGBw22D8bHgaP.vGnIggM7YVj7LHE7Muc9bwRsH7TclLKTywQ6tmMF8hT7Z6XsXmqM5WRpL2ENY3u4pyXoQBxiRITx%7C1%7C-1','cmt_vin':'e754154fe4a567dd44864ffe1658fa37e24258408af0e2e319bbe3bbccdfb0f2'}

r = post(url, cookies=cookies,data=data)

print r.content.decode('utf8')
```

最后一行为，打印出返回的内容
成功截图

{% asset_img  7.jpg %}

结束语：如果想自动顶帖，写个循环就可以啦，不过用同一个用户，容易被封，多搞几个cookie就行了嘛，也不要一直用同一句话，可以做个字典啊，随机选一句话，随机选一个用户，然后发表，最后时间也可以随机选，那样被服务器检测到的概率就大大降低了~~yes~完成。