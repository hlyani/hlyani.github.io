# pyecharts 相关

##### 1、爬取链家使用pyecharts显示

```
# coding: utf-8

import urllib2 as ulib
import re
import pandas as pd
import numpy as np
import json
from pyecharts import Bar
from pyecharts import Line

url = "https://cd.lianjia.com/ershoufang/"

headers = {'User-Agent':'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.62 Safari/537.36'}
r = ulib.Request(url,headers = headers)
data = ulib.urlopen(r)
data = data.read().decode('utf-8')

all_re = '<div class="item"([\s\S]*?)<div class="item"'
price_re = 'price"><span>([\s\S]*?)</span>'
name_re = 'data-el="ershoufang">([^<][\s\S]*?)</a>'
other_re = 'info">([\s\S]*?)</div>'

all_infos = re.findall(all_re, data)
price_infos = []
name_infos = []
other_infos = []
for info in all_infos:
    price = re.findall(price_re,info)[0]
    price_infos.append(price)
    name = re.findall(name_re,info)[0]
    name_infos.append(name)
    other = re.findall(other_re,info)[0].replace('<span>/</span>','|')
    other_infos.append(other)

house = pd.DataFrame({'name':name_infos,'price':price_infos,'other':other_infos})

#对房源信息进行分列
other_split = pd.DataFrame((x.split('|') for x in house.other),index=house.index,columns=['address','house_type','area','direction','decoration','elevator'])
house = pd.merge(house,other_split,left_index=True, right_index=True)
house.drop(house.columns[1],axis=1,inplace=True)
house['area'].replace('[^0-9.]', '',regex=True,inplace=True)
print(house)
data={"name":list(house["name"]),"price":list(house["price"].astype('int')),"area":list(house["area"])}

# # pip install pyecharts

bar = Line("house info", "这里是副标题")
price_array = house["price"].astype('int')
area_array = house["area"].astype('float')

# bar.add("price",range(len(price_array)), price_array)

bar.add("price",list(house["name"]), price_array)
bar.add("area",list(house["name"]), area_array)

# # bar.print_echarts_options() # 该行只为了打印配置项，方便调试时使用

bar.render()    # 生成本地 HTML 文件
```

