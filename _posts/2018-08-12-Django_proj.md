---
layout:     post
title:      "利用Django和pyecharts将AQI可视化"
subtitle:   "初步学习Django使用"
date:       2018-08-11
author:     dengzhenzhen
header-img: img/post-bg-Django.png
catalog: true
tags:
    - Django
    - web   
---

# 前言

## 0.1 为什么要做这个
Django是一个广泛使用的python web框架，有了它搭建一个网站后台简直轻而易举。不过对于一个没接触过web开发初学者(也就是我)来说，刚开始看到那一大堆文件头都大了，哪个文件是用来干嘛的都不知道。

接触Django以来，一共做过两个简单的项目：
- 官方文档给的[投票项目](URL https://docs.djangoproject.com/zh-hans/2.0/intro/tutorial01/)
- AQI信息可视化：[AQI_map](URL https://github.com/dengzhenzhen/AQI_map)

对着文档敲完第一个项目的时候，其实还是懵懂的，很多东西稀里糊涂就过去了，也就是对项目里包含的文件有个大概的认识。所以之后又对着文档做了第二个空气质量指数可视化。所以做这个东西纯粹是为了学习。

## 0.2 AQI_map是什么
AQI,AKA air quality index,空气质量指数，是用来定量描述空气质量状况的指数，越大空气质量越差。我们常关注的PM2.5浓度就是AQI计算的其中一项。
AQI_map将全国主要城市的AQI显示在地图上，每个城市的点大小和AQI的值有关。

## 0.3 用到的工具
- python环境 : anaconda3 4.3.0
- Django版本 : 2.0.7
- AQI信息来源 : 爬虫爬取pm25.in排行榜
- 地理位置信息来源 : 百度地图API


```python
!python --version
```

    Python 3.6.0 :: Anaconda 4.3.0 (64-bit)
    


```python
import django
django.get_version()
```




    '2.0.7'



# 实现过程

## 1.1 创建项目
cd到用来存放项目的目录

运行命令


```python
#!django-admin startproject AQI_map
```

运行后会得到一个名为AQI的文件夹，文件夹存放的是项目的文件，名字可以随便改

文件夹包含一个 *manage.py* 文件和一个名为 *AQI* 的文件夹

接下来cd到工程的目录 */AQI* ，运行命令创建app


```python
#python manage.py startapp AQI
```

运行命令之后，将会创建一个新的文件夹* AQI*，文件夹包含如下内容:

-    \__init__.py
-   admin.py
-   apps.py
-   migrations/\__init__.py
-   models.py
-   tests.py
-   views.py

此外，还需要建立一个 *urls.py*

- urls.py

## 1.2 urls.py

在项目文件夹 */AQI* 和app文件夹* /AQI_map* 里都存在 *urls.py* 

用来指定某个url对应某个视图


```python
#/AQI/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('index', views.index, name='index'),
]
```


```python
#/AQI/urls.py

from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('AQI/', include('AQI.urls'))
]
```

这样，当我们在浏览器输入 *'host/AQI/index'* 时，先经过 */AQI/urls.py* 处理，将url截断为 *'index'*,传入 AQI/urls.py处理

在 */AQI/urls.py* 中，'index'与urlpatterns中的某个path匹配，于是调用了 *views.index*

## 1.3 views.py

views.py长这样：


```python
#/AQI/views.py

from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world!")
```

当接收到一个请求时，从 *urls.py *映射到* views.index* 于是返回一个response

此时，万里长征的第一步就算完成了！

### 1.31 .html和.js

我们的前端网页是用html和js做的，我们当然想要返回这些文件

其实就算直接用


```python
open('xxx.html').read()
```

读出一个str，再返回这个str也不是不可以

不过这样不优雅还很麻烦，再说这样的话我们还用框架干嘛

所以应该这样：

- 在app文件夹下创建 */templates/AQI* 和 */static/AQI *
- 前者用来放html，后者放js
- 我的文件名为effectScatter-bmap.html  echarts.min.js
- 在html文件加上：


```python
{% load static %}
<script type="text/javascript" src="{% static 'AQI/echarts.min.js' %}"></script>
```

这样html就能加载js脚本了

### 1.32 render

前面的做完后，可以用render函数来返回


```python
from django.shortcuts import render
def index(request):
    context={}
    return render(request, 'AQI/effectScatter-bmap.html',context)
```

context用来将值从python传递到html里

## 1.4 models

其实models就是用来定义数据库用什么表，表有什么字段

### 1.41 设计表


```python
#/AQI/models.py

from django.db import models

class AQI_data(models.Model):
    CITY = models.CharField(max_length=10)
    AQI = models.IntegerField()
    TIME = models.DateTimeField(auto_now_add=True)

class geo_data(models.Model):
    CITY = models.TextField(primary_key=True)
    CRD = models.TextField()
```

上面代码定义了两张表AQI_data和geo_data

实际在数据库中的表名为AQI_aqi_data和AQI_geo_data

在settings.py中可以指定用的数据库类型


```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

因为项目很简单所以就用的sqlite3，其他数据库还没研究

### 1.42 migrate

models改动之后，需要运行命令来把改动应用到数据库中：



```python
!python manage.py makemigrations AQI
```


```python
!python manage.py migrate
```

migrate之后，表的修改就完成了，可以进行增删查改blablabla

django中不需要使用数据库语句，可以直接用models里的类来操作

## 1.5 获取数据

### 1.51 AQI数据
AQI数据从[pm25.in排行榜](URL http://www.pm25.in/rank)爬取


```python
def AQI_info():
    '''
    return value format:
    [
        [rank, city, AQI, MARK, PRIOTY, PM25, PM10, CO, NO2, O3, O3_AVG, SO2, TIME, POSITION],
        ...,
        ...,
        [rank, city, AQI, MARK, PRIOTY, PM25, PM10, CO, NO2, O3, O3_AVG, SO2, TIME, POSITION]
    ]
    '''
    url = 'http://www.pm25.in/rank'
    headers = { 'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36'}
    req = requests.get(url, headers=headers)
    soup = bs4.BeautifulSoup(req.text, 'lxml')
    
    result_lst = [ [label.text for label in i.find_all('td')]  for i in soup.find('tbody').find_all('tr')]
    return result_lst
```

### 1.52 地理位置信息

要在地图上显示还需要有经纬度坐标

这里使用了[百度地图API](URL http://lbsyun.baidu.com/index.php?title=%E9%A6%96%E9%A1%B5)获取坐标

要使用API需要先注册获得一个**AK**


```python
def geo_info(city_name):
    '''
    return value format:
    {'city': '赣州', 'geo_info': ['114.935909079', '25.8452955363']}
    '''
    addr = city_name
    geo_url = 'http://api.map.baidu.com/geocoder/v2/?address={0}output=json&ak={1}'.format(addr,ak)
    req = requests.get(geo_url)
    soup = bs4.BeautifulSoup(req.text, 'lxml')
    return {'city':city_name, 'geo_info':[float(soup.lng.text), float(soup.lat.text)]}
```

### 1.6 完整实现

接收到请求先看数据库是否有一小时内的内容

有的话读取数据库中的内容

没有的话运行爬虫爬取pm25.in的排行榜数据并存入表AQI_data中

读取到城市、AQI后，从表geo_data读取经纬度信息

如果经纬度信息不存在，用百度地图API获取该城市经纬度


```python
#views.py

from django.shortcuts import render
from AQI.models import AQI_data,geo_data
from AQI.functions import AQI_info,geo_info
from django.utils import timezone
from django.db.models import Q
import json

def index(request):

    now = timezone.datetime.now()
    start = now - timezone.timedelta(hours=0,minutes=59,seconds=59)
    result = []

    a = AQI_data.objects.filter(Q(TIME__range=(start, now) ))
    if a.count() == 0:
        AQI_present = AQI_info()
        for i in AQI_present:
            a = AQI_data()
            a.CITY = i[1]
            a.AQI = i[2]
            try:
                a.save()
                result.append(i[1:3])
            except:
                continue    
    else:
        result = [ [query.CITY, query.AQI] for query in a]

    for i in result:
        try:
            a = geo_data.objects.get(CITY=i[0])
            i.append(a.CRD)
        except:
            a = geo_data()
            a.CITY = i[0]
            try:
                a.CRD = geo_info(i[0])['geo_info']
            except:
                a.CRD = 'not exist'
            i.append(a.CRD)
            a.save()
    data = [{'name':i[0], 'value':i[1]} for i in result]
    geoCoordMap = {}
    for i in result:
        geoCoordMap[i[0]] = i[2]

    DATA = json.dumps(data, ensure_ascii=False).replace('"name"','name').replace('"value"','value')
    GEOMAP = json.dumps(geoCoordMap, ensure_ascii=False).replace('"[','[').replace(']"',']')

    context = dict(DATA=DATA, GEOMAP=GEOMAP)
    #return HttpResponse(template.render(context, request))
    return render(request, 'AQI/effectScatter-bmap.html',context)
```


```python
#effectScatter-bmap.html
...
...
var data = {{ DATA|safe }};
var geoCoordMap = {{ GEOMAP|safe }};

...
...
```

效果大概是这样
![图片](https://s1.ax1x.com/2018/08/12/PcwCdA.png)

# 总结

实践总是最好的学习方法，敲第一个投票项目时只是跟着文档的代码敲，没什么自己的思考，导致并没有什么理解

做AQI地图的时候边查文档边敲代码边理解，所以对django的了解也更深入些了
