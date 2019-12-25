# Jinja2 相关

>  jinja2 是 Flask 作者开发的一个模板系统 

[jinja2 官网](https://jinja.palletsprojects.com/en/2.10.x/)

### 一、安装

##### 1、安装jinja2

```
pip install jinja2
```

##### 2、测试是否安装成功

```
python -c "import jinja2"
```

### 二、基本语法

* 控制结构

{% raw %} 

​		{% %}

{% endraw %} 

* 变量取值

{% raw %} 

​		{{ }}

{% endraw %} 

* 注释

{% raw %} 

​		{# #}

{% endraw %} 

例子：

```
{# This is jinja code
    {% for file in filenames %}
    ...
    {% endfor %}
#}
```

>  可以看到，for循环的使用方式和Python比较类似，但是没有了句尾的冒号，另外需要使用endfor最为结尾，其实在jinja2中，if也是一样的，结尾需要使用endif。 

##### 1、变量

```
<p>this is a dicectory:{{ mydict['key'] }} </p>
<p>this is a list:{{ mylist[3] }} </p>
<p>this is a object:{{ myobject.something() }} </p>
```

##### 2、过滤器

>  变量可以通过“过滤器”进行修改，过滤器可以理解为是jinja2里面的内置函数和字符串处理函数。 

| 常用过滤器  | 说明                                         |
| ----------- | -------------------------------------------- |
| safe        | 渲染时值不转义                               |
| capitialize | 把值的首字母转换成大写，其他子母转换为小写   |
| lower       | 把值转换成小写形式                           |
| upper       | 把值转换成大写形式                           |
| title       | 把值中每个单词的首字母都转换成大写           |
| trim        | 把值的首尾空格去掉                           |
| striptags   | 渲染之前把值中所有的HTML标签都删掉           |
| join        | 拼接多个值为字符串                           |
| replace     | 替换字符串的值                               |
| round       | 默认对数字进行四舍五入，也可以用参数进行控制 |
| int         | 把值转换成整型                               |

```
{{ 'abc' | captialize  }}
# Abc
 
{{ 'abc' | upper  }}
# ABC
 
{{ 'hello world' | title  }}
# Hello World
 
{{ "hello world" | replace('world','daxin') | upper }}
# HELLO DAXIN
 
{{ 18.18 | round | int }}
# 18
```

##### 3、控制结构

```
{% if daxin.safe %}
daxin is safe.
{% elif daxin.dead %}
daxin is dead
{% else %}
daxin is okay
{% endif %}
```

##### 4、for 循环

```
<title>{% block title %}{% endblock %}</title>
<ul>
{% for user in users %}
  <li><a href="{{ user.url }}">{{ user.username }}</a></li>
{% endfor %}
</ul>
```

> 循环生成<li></li>

```
<ul>
    {% for user in users %}
    <li>{{ user.username|title }}</li>
    {% endfor %}
</ul>
```

> 迭代字典

```
<dl>
    {% for key, value in my_dict.iteritems() %}
    <dt>{{ key }}</dt>
    <dd>{{ value}}</dd>
    {% endfor %}
</dl>
```

>  在for循环中，jinja2还提供了一些特殊的变量，用以来获取当前的遍历状态 

| 变量           | 描述                         |
| -------------- | ---------------------------- |
| loop.index     | 当前迭代的索引（从1开始）    |
| loop.index0    | 当前迭代的索引（从0开始）    |
| loop.first     | 是否是第一次迭代，返回bool   |
| loop.last      | 是否是最后一次迭代，返回bool |
| loop.length    | 序列中的项目数量             |
| loop.revindex  | 到循环结束的次数（从1开始）  |
| loop.revindex0 | 到循环结束的次数(从0开始）   |

##### 5、继承和super函数

>  base.html 

```
<!DOCTYPE html>
<html lang="en">
<head>
    {% block head %}
    <link rel="stylesheet" href="style.css"/>
    <title>{% block title %}{% endblock %} - My Webpage</title>
    {% endblock %}
</head>
<body>
<div id="content">{% block content %}{% endblock %}</div>
<div id="footer">
    {% block  footer %}
    <script>This is javascript code </script>
    {% endblock %}
</div>
</body>
</html>
```

> 使用继承， 这里定义了四处 block，即：head，title，content，footer。那怎么进行继承和变量替换呢？注意看下面的文件 

```
{% extend "base.html" %}       # 继承base.html文件
 
{% block title %} Dachenzi {% endblock %}   # 定制title部分的内容
 
{% block head %}
    {{  super()  }}        # 用于获取原有的信息
    <style type='text/css'>
    .important { color: #FFFFFF }
    </style>
{% endblock %}   
 
# 其他不修改的原封不动的继承
```

### 三、基本使用方法

```
>>> from jinja2 import Template
>>> template = Template('Hello {{ name }}!')
>>> template.render(name='John Doe')
u'Hello John Doe!'
```

大多数应用都在初始化的时候撞见一个Environment对象，并用它加载模板。Environment支持两种加载方式：

* PackageLoader：包加载器

- FileSystemLoader：文件系统加载器

```
from jinja2 import PackageLoader,Environment
env = Environment(loader=PackageLoader('python_project','templates')) # 创建一个包加载器对象
 
template = env.get_template('bast.html')    # 获取一个模板文件
template.render(name='daxin',age=18)   # 渲染
```

- PackageLoader()的两个参数为：python包的名称，以及模板目录名称。
- get_template()：获取模板目录下的某个具体文件。
- render()：接受变量，对模板进行渲染