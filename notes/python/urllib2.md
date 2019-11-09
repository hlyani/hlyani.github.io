# urllib2 爬虫

```
import urllib2
import re
import os

url = r'https://movie.douban.com/cinema/nowplaying/chengdu/'
req = urllib2.Request(url, headers={'User-Agent': 'Magic Browser'})
webpage = urllib2.urlopen(req)
html = webpage.read()

# html.decode('utf-8')

re_general = r'<div id="nowplaying">([\s\S]*?)<div id="upcoming">'

general = re.findall(re_general, html)[0]

tg_start = 0
tg_end = 0
names = []
rates = []

tg_start = general.count('<ul class="">')

for i in range(34):
    tg_start = general.find('<ul class="">')
    # if tg_start == -1:
    #     print("not find start tag")
    #     os.exit()
    tmp = general[tg_start:-1]
    general = tmp[len('<ul class="">'):-1]
    tg_end = tmp.find("</ul>")
    # if tg_end == -1:
    #     print("not find end tag")
    #     os.exit()
    tmp = tmp[len('<ul class="">'):tg_end]

    re_name = r'"title">([\s\S]*?)</a>'
    re_rate = r'"subject-rate">([\s\S]*?)</span>'
    re_no_rate = r'"text-tip">([\s\S]*?)</span>'
    name = re.findall(re_name, tmp)[0].strip()

    # rate = re.findall(re_rate, tmp)[0].strip() if re.findall(
    #     re_rate, tmp) else re.findall(re_no_rate, tmp)[0].strip()

    rate = re.findall(re_rate, tmp)[0].strip() if re.findall(
        re_rate, tmp) else "0"

    names.append(name)
    rates.append(float(rate))
    print(name + ":" + rate)

print(names)
print(rates)


def get_avarage(num_list):
    nsum = 0
    for i in range(len(num_list)):
        nsum += i
    return nsum/len(num_list)


film_num = len(names)
max_rate = max(rates
index_max_rate = rates.index(max_rate)
avarage_rate = get_avarage(rates)
print("film sum: %d" % film_num)
print("max rate: %f,film name is: %s" % (max_rate, names[index_max_rate]))
print("avarage rate: %.2f" % avarage_rate)
```

