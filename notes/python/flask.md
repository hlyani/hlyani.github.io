# Flask 相关

[flask 官网文档]( https://dormousehole.readthedocs.io/en/latest/ )

[Flask 教程]( https://www.w3cschool.cn/flask/ ) 

### 一、准备

##### 1、安装

```
pip install Flask
```

##### 2、测试

```
python -c "import flask"

flask --version
```

##### 3、一般flask项目的目录结构

```
flask-demo/
  ├ run.py           # 应用启动程序
  ├ config.py        # 环境配置
  ├ requirements.txt # 列出应用程序依赖的所有Python包
  ├ tests/           # 测试代码包
  │   ├ __init__.py 
  │   └ test_*.py    # 测试用例
  └ myapp/
      ├ admin/       # 蓝图目录
      ├ static/
      │   ├ css/     # css文件目录
      │   ├ img/     # 图片文件目录
      │   └ js/      # js文件目录
      ├ templates/   # 模板文件目录
      ├ __init__.py    
      ├ forms.py     # 存放所有表单，如果多，将其变为一个包
      ├ models.py    # 存放所有数据模型，如果多，将其变为一个包
      └ views.py     # 存放所有视图函数，如果多，将其变为一个包
```

### 二、使用

##### 1、新建文件 hello.py ，并输入以下内容

```
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
	app.debug = True # 设置调试模式，生产模式的时候要关掉debug
	app.run(host="0.0.0.0")
```

##### 2、运行

```
python hello.py

 * Running on http://0.0.0.0:5000/
```

> 在浏览器中访问 IP:5000
>
> IP 为你的虚拟机地址，例：
>
> http://192.168.1.1:5000

##### 3、模板，flask基于jinja2模板引擎实现

[jinja2 相关](https://hlyani.github.io/notes/python/jinja2.html)

> 编辑 hello.html

{% raw %}
```
<!doctype html>
<title>Hello Sample</title>

{% if name %}
<h1>Hello {{ name }}!</h1>
{% else %}
<h1>Hello World!</h1>
{%  endif  %}
```
{% endraw %}

> 编辑 hello.py

```
from  flask import  Flask
from  flask import  render_template

app  =  Flask(__name__)
@app.route('/hello')
@app.route('/hello/<name>')

def  hello(name=None):
    return  render_template('hello.html',  name=name)

if  __name__  ==  '__main__':
    app.run(host='0.0.0.0',  debug=True)
```

> 浏览器访问： http://192.168.1.1:5000/hello/man 

##### 4、模板继承

> 新建，并编辑 layout.html 的模板

{% raw %}
```
<!doctype html>

<title>Hello Sample</title>

<link rel="stylesheet"  type="text/css"  href="{{ url_for('static', filename='style.css') }}">

<div class="page">
{% block body %}
{% endblock %}
</div>
```
{% endraw %}

> 修改之前的 hello.html

{% raw %}
```
{%  extends  "layout.html" %}
{%  block  body  %}
{%  if  name  %}
<h1>Hello {{ name }}!</h1>
{% else %}
<h1>Hello World!</h1>
{%  endif  %}
{%  endblock  %}
```
{% endraw %}

> 浏览器访问： http://192.168.1.1:5000/hello/man ，查看源码

##### 5、HTML自动转义

> 打开页面，你会看到”Hello Flask”字样，而且”Flask”是斜体的，因为我们加了”em”标签。但有时我们并不想让这些HTML标签自动转义，特别是传递表单参数时，很容易导致HTML注入的漏洞。我们把上面的代码改下，引入”Markup”类：

```
from  flask import  Flask,  Markup

app  =  Flask(__name__)

@app.route('/')

def  index():
    return  Markup('<div>Hello %s</div>')  %  '<em>Flask</em>'
```

##### 6、request 对象

#####  layout.html

{% raw %}
```
<!doctype html>
<title>Hello Sample</title>
<link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='style.css') }}">
<div class="page">
{% block body %}
{% endblock %}
</div>
```
{% endraw %}

##### login.html

{% raw %}
```
{% extends "layout.html" %}
{% block body %}
<form name="login" action="/login" method="post">
Hello {{ title }}, please login by:
<input type="text" name="user" />
</form>
{% endblock %}
```
{% endraw %}

```
from flask import Flask,url_for,request,render_template

@app.route('/login', methods=['POST', 'GET'])
def login():
    if request.method == 'POST':
        if request.form['user'] == 'admin':
            return 'Admin login successfully!'
        else:
            return 'No such user!'
    title = request.args.get('title', 'Default')
    return render_template('login.html', title=title)
if __name__ == "__main__":
    app.run(debug=True)
```

> request中”method”变量可以获取当前请求的方法，即”GET”, “POST”, “DELETE”, “PUT”等；”form”变量是一个字典，可以获取Post请求表单中的内容，在上例中，如果提交的表单中不存在”user”项，则会返回一个”KeyError”，可以不捕获，页面会返回400错误（想避免抛出这”KeyError”，可以用request.form.get(“user”)来替代）。而”request.args.get()”方法则可以获取Get请求URL中的参数，该函数的第二个参数是默认值，当URL参数不存在时，则返回默认值。
>
>  在当前目录下，创建一个子目录”templates”（注意，一定要使用这个名字）。然后在”templates”目录下，添加 

##### 7、会话 session

```
from flask import Flask,url_for,request,render_template,redirect,session
@app.route('/login', methods=['POST', 'GET'])
def login():
    if request.method == 'POST':
        if request.form['user'] == 'admin':
            session['user'] = request.form['user']
            return 'Admin login successfully!'
        else:
            return 'No such user!'
    if 'user' in session:
        return 'Hello %s!' % session['user']
    else:
        title = request.args.get('title', 'Default')
        return render_template('login.html', title=title)
@app.route('/logout')
def logout():
    session.pop('user', None)
    return redirect(url_for('login'))

app.secret_key = '123456'
if __name__ == "__main__":
    app.run(debug=True)
```

> 可以看到，”admin”登陆成功后，再打开”login”页面就不会出现表单了。然后打开logout页面可以登出。session对象的操作就跟一个字典一样。特别提醒，使用session时一定要设置一个密钥”app.secret_key”，如上例。不然会得到一个运行时错误，内容大致是”RuntimeError: the session is unavailable because no secret key was set”。密钥要尽量复杂，最好使用一个随机数，这样不会有重复，上面的例子不是一个好密钥。

##### 8、构建响应

>  在之前的例子中，请求的响应都是直接返回字符串内容，或者通过模板来构建响应内容然后返回。其实也可以先构建响应对象，设置一些参数（比如响应头）后，再将其返回。修改下上例中的Get请求部分： 

```
from flask import Flask,url_for,request,render_template,redirect,session,make_response

@app.route('/login', methods=['POST', 'GET'])
def login():
    if request.method == 'POST':
        ...
    if 'user' in session:
        ...
    else:
        title = request.args.get('title', 'Default')
        response = make_response(render_template('login.html', title=title), 200)
        response.headers['key'] = 'value'
        return response
if __name__ == "__main__":
    app.run(debug=True)
```

##### 9、Cookie的使用

```
from flask import Flask,url_for,request,render_template,redirect,session,make_response
import time
@app.route('/login', methods=['POST', 'GET'])
def login():
    response = None
    if request.method == 'POST':
        if request.form['user'] == 'admin':
            session['user'] = request.form['user']
            response = make_response('Admin login successfully!')
            response.set_cookie('login_time', time.strftime('%Y-%m-%d %H:%M:%S'))
        ...

    else:
        if 'user' in session:
            login_time = request.cookies.get('login_time')
            response = make_response('Hello %s, you logged in on %s' % (session['user'], login_time))
        ...

    return response
app.secret_key = '123456'
if __name__ == "__main__":
    app.run(debug=True)
```

##### 10、错误处理

```
from flask import Flask,abort

app = Flask(__name__)
@app.route('/error')
def error():
    abort(404)

if __name__ == "__main__":
    app.run(debug=True)
```

```
from flask import Flask,abort

app = Flask(__name__)

@app.errorhandler(404)
def page_not_found(error):
    return render_template('404.html'), 404
if __name__ == "__main__":
    app.run(debug=True)
```

> 在实际开发过程中，并不会经常使用”abort()”来退出，常用的错误处理方法一般都是异常的抛出或捕获。装饰器”@app.errorhandler()”除了可以注册错误代码外，还可以注册指定的异常类型。让我们来自定义一个异常：

```
class InvalidUsage(Exception):
    status_code = 400

    def __init__(self, message, status_code=400):
        Exception.__init__(self)
        self.message = message
        self.status_code = status_code


@app.errorhandler(InvalidUsage)
def invalid_usage(error):
    response = make_response(error.message)
    response.status_code = error.status_code
    return response
@app.route('/exception')
def exception():
    raise InvalidUsage('No privilege to access the resource', status_code=403)
if __name__ == "__main__":
    app.run(debug=True)
```

> 在上面的代码中定义了一个异常”InvalidUsage”，同时通过装饰器”@app.errorhandler()”修饰了函数”invalid_usage()”，装饰器中注册了我们刚定义的异常类。这也就意味着，一但遇到”InvalidUsage”异常被抛出，这个”invalid_usage()”函数就会被调用。

##### 11、url 重定向

>  Flask的URL规则是基于Werkzeug的路由模块。模块背后的思想是基于 Apache 以及更早的 HTTP 服务器主张的先例，保证优雅且唯一的 URL。 

```
@app.route('/projects/')
def projects():
    return 'The project page'

@app.route('/about')
def about():
    return 'The about page'
```

> 访问第一个路由不带/时，Flask会自动重定向到正确地址。
>  访问第二个路由时末尾带上/后Flask会直接报404 NOT FOUND错误。
>
> 重定向”redirect()”函数的使用在上面例子中已有出现。作用就是当客户端浏览某个网址时，将其导向到另一个网址。常见的例子，比如用户在未登录时浏览某个需授权的页面，将其重定向到登录页要求其登录。

```
from flask import session, redirect
 
@app.route('/')
def index():
    if 'user' in session:
        return 'Hello %s!' % session['user']
    else:
        return redirect(url_for('login'), 302)
```

