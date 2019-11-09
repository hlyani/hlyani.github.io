# scrapy 爬虫

[scrapy官网](https://scrapy-chs.readthedocs.io/zh_CN/latest/intro/overview.html)

```
scrapy.cfg: 项目的配置文件
tutorial/: 该项目的python模块。之后您将在此加入代码。
tutorial/items.py: 项目中的item文件.
tutorial/pipelines.py: 项目中的pipelines文件.
tutorial/settings.py: 项目的设置文件.
tutorial/spiders/: 放置spider代码的目录.
```

- Item 是保存爬取到的数据的容器
- Spider是用户编写用于从单个网站(或者一些网站)爬取数据的类

##### 为了创建一个Spider，您必须继承 scrapy.Spider 类， 且定义以下三个属性:

- name: 用于区别Spider。 该名字必须是唯一的，您不可以为不同的Spider设定相同的名字。
- start_urls: 包含了Spider在启动时进行爬取的url列表。 因此，第一个被获取到的页面将是其中之一。 后续的URL则从初始的URL获取到的数据中提取。
- parse() 是spider的一个方法。 被调用时，每个初始URL完成下载后生成的 Response 对象将会作为唯一的参数传递给该函数。 该方法负责解析返回的数据(response data)，提取数据(生成item)以及生成需要进一步处理的URL的 Request 对象。

##### Selector有四个基本的方法(点击相应的方法可以看到详细的API文档):

- xpath(): 传入xpath表达式，返回该表达式所对应的所有节点的selector list列表 。
- css(): 传入CSS表达式，返回该表达式所对应的所有节点的selector list列表。
- extract(): 序列化该节点为unicode字符串并返回list。
- re(): 根据传入的正则表达式对数据进行提取，返回unicode字符串list列表。

### 一、基本使用

##### 1、安装

```
pip install scrapy
```

##### 2、查看所有命令

```
scrapy genspider -l
```

##### 3、查看模板命令

```
scrapy genspider -d 模板名称
```

##### 4、展示爬虫应用列表

```
scrapy list
```

##### 5、创建scrapy项目

```
scrapy startproject yani
cd yani/
```

##### 6、生成 spider

```
scrapy genspider hotel "https://hotels.ctrip.com/hotel/"
```

##### 7、运行单独爬虫应用

> scrapy crawl 爬虫应用名称

```
scrapy crawl hotel
```

```
scrapy crawl hotel -o output.json
```

##### 8、示例代码

[ https://github.com/hlyani/myscrapy ](https://github.com/hlyani/myscrapy)

```
git clone https://github.com/hlyani/myscrapy.git
```

##### 9、scrapy shell

```
scrapy shell 'http://scrapy.org' --nolog

response.xpath('//title/text()').re('(\w+):')
```

### 二、节点匹配相关

##### 1、xpath

[xpath](https://hlyani.github.io/notes/python/xpath.html)

##### 2、正则

```
response.xpath("//span[contains(@class, 'bookmark-btn')]/text()").re('.*?(\d+).*')
response.css(".bookmark-btn::text").re('.*?(\d+).*')
response.xpath('//div[@class="quote"]/span[@class="text"]/text()').re("Harry, (\w.*)far more than")
```

##### 3、css

```
response.css("a::attr(href)")
response.css(".page-navigator a::attr(href)").extract()
response.css(".post-content img::attr(src)").extract()
response.css("p::text").extract()
response.css("title::text").extract()
response.css("div::text").extract()
response.css(".center::text").extract()
response.css(".quote span::text").getall()
response.css(".quote span::text").getall()
response.xpath('//small[@class="author"]/text()').getall()
response.xpath('//div[@class="quote"]/span[@class="text"]/text()').re("Harry, (\w.*)far more than")
```

