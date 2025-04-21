# Flask 相关

[flask 官网文档]( https://dormousehole.readthedocs.io/en/latest/ )

[Flask 教程]( https://www.w3cschool.cn/flask/ ) 

# 一、准备

## 1、安装

```
pip install Flask
```

## 2、测试

```
python -c "import flask"
flask --version
```

## 3、一般flask项目的目录结构

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

# 二、使用

## 1、新建文件 hello.py ，并输入以下内容

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

## 2、运行

```
python hello.py

 * Running on http://0.0.0.0:5000/
```

> 在浏览器中访问 IP:5000
>
> IP 为你的虚拟机地址，例：
>
> http://192.168.1.1:5000

## 3、模板

flask基于jinja2模板引擎实现

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

## 4、模板继承

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

## 5、HTML自动转义

> 打开页面，你会看到”Hello Flask”字样，而且”Flask”是斜体的，因为我们加了”em”标签。但有时我们并不想让这些HTML标签自动转义，特别是传递表单参数时，很容易导致HTML注入的漏洞。我们把上面的代码改下，引入”Markup”类：

```
from  flask import  Flask,  Markup

app  =  Flask(__name__)

@app.route('/')

def  index():
    return  Markup('<div>Hello %s</div>')  %  '<em>Flask</em>'
```

## 6、request 对象

##### layout.html

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

## 7、会话 session

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

## 8、构建响应

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

## 9、Cookie的使用

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

## 10、错误处理

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

## 11、url 重定向

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

## 12、配置热加载

1. watchdog

```
pip install watchdog

import json
from watchdog.observers import Observer
from watchdog.events import FileSystemEventHandler

CONFIG_PATH = "config.json"
config = {}

class ConfigHandler(FileSystemEventHandler):
    def on_modified(self, event):
        if event.src_path.endswith(CONFIG_PATH):
            load_config()

def load_config():
    global config
    with open(CONFIG_PATH, "r") as f:
        config = json.load(f)
    print("配置已更新:", config)

# 监听配置文件变更
event_handler = ConfigHandler()
observer = Observer()
observer.schedule(event_handler, ".", recursive=False)
observer.start()

try:
    while True:
        pass
except KeyboardInterrupt:
    observer.stop()
observer.join()
```

2. importlib.reload()

```
import importlib
import config  # 假设配置在 config.py 中

def reload_config():
    global config
    importlib.reload(config)
    print("配置已重新加载:", config.settings)  # 例如 config.py 里有 settings 变量

# 调用 reload_config() 即可刷新配置
```

3. inotify-tools

```
apk add inotify-tools  # Alpine
apt-get update && apt-get install -y inotify-tools  # Debian/Ubuntu

while inotifywait -e modify /conf/app.conf; do
    echo "Config file changed, reloading..."
    # 在这里可以执行重载命令，比如：
    # nginx -s reload
done

```

```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app-container
    image: my-app
    volumeMounts:
    - name: config-volume
      mountPath: /conf
  - name: reload-sidecar
    image: busybox
    command: [ "sh", "-c", "while inotifywait -e modify /conf/app.conf; do echo 'Reloading...'; kill -HUP 1; done" ]
    volumeMounts:
    - name: config-volume
      mountPath: /conf
  volumes:
  - name: config-volume
    configMap:
      name: my-config	
```

## 13、gunicorn

> gunicorn_config.py

```
from gevent import monkey
monkey.patch_all()

from api.utils.log_utils import initRootLogger
initRootLogger("ragflow_server")

import logging
import gevent
import signal

from api import settings
from api.db.runtime_config import RuntimeConfig
from api.db.services.document_service import DocumentService
from api import utils
from gevent.event import Event

from api.db.db_models import init_database_tables as init_web_db
from api.db.init_data import init_web_data
from api.versions import get_ragflow_version
from api.utils import show_configs
from rag.settings import print_rag_settings
from rag.utils.redis_conn import RedisDistributedLock


stop_event = Event()

def update_progress():
    redis_lock = RedisDistributedLock("update_progress", timeout=60)
    while not stop_event.is_set():
        acquired = False
        try:
            acquired = redis_lock.acquire()
            if not acquired:
                gevent.sleep(2)
                continue
            DocumentService.update_progress()
            gevent.sleep(6)
        except Exception as e:
            logging.exception("update_progress exception: %s", e)
        finally:
            if acquired:
                try:
                    redis_lock.release()
                except Exception as e:
                    logging.warning("Failed to release Redis lock: %s", e)

def start_prometheus_server():
    logging.info("promethues HTTP server start...")
    from prometheus_client import start_http_server as prometheus_start_http_server
    # TODO settings.METRICS_PORT
    try:
        prometheus_start_http_server(8000)
    except Exception as e:
        logging.error("Failed to start promethues: %s", e)

def start_background_task():
    gevent.spawn(update_progress)
    gevent.spawn(start_prometheus_server)

def signal_handler(signum, frame):
    logging.info(f"Received signal {signum}, shutting down...")
    if not stop_event.is_set():
        stop_event.set()

def register_nacos_server():
    if settings.NACOS.get("address"):
        logging.info("Regist to nacos start...")
        import nacos
        try:
            address = settings.NACOS.get("address")
            namespace = settings.NACOS.get("namespace")
            username = settings.NACOS.get("username")
            password = settings.NACOS.get("password")
            instance_ip = settings.NACOS.get("instance_ip")
            instance_port = settings.NACOS.get("instance_port")

            client = nacos.NacosClient(
                address, namespace=namespace, username=username, password=password
            )
            client.add_naming_instance(
                "ragflow", instance_ip, instance_port,
                cluster_name="DEFAULT", ephemeral=True, heartbeat_interval=5
            )
            logging.info("Successfully registered to Nacos")
        except Exception as e:
            logging.error("Failed to register to Nacos: %s", e)

def on_starting(server):
	pass

def post_fork(server, worker):
    settings.init_settings()
    RuntimeConfig.init_env()
    RuntimeConfig.init_config(JOB_SERVER_HOST=settings.HOST_IP, HTTP_PORT=settings.HOST_PORT)
    if not server.WORKERS:  # 只有主进程的 worker的 server.WORKERS 为 {}
        logging.info(r"""
            ____   ___    ______ ______ __               
        / __ \ /   |  / ____// ____// /____  _      __
        / /_/ // /| | / / __ / /_   / // __ \| | /| / /
        / _, _// ___ |/ /_/ // __/  / // /_/ /| |/ |/ / 
        /_/ |_|/_/  |_|\____//_/    /_/ \____/ |__/|__/                             

        """)
        logging.info(
            f'RAGFlow version: {get_ragflow_version()}'
        )
        logging.info(
            f'project base: {utils.file_utils.get_project_base_directory()}'
        )
        show_configs()
        print_rag_settings()
        init_web_db()
        init_web_data()
        register_nacos_server()
        start_background_task()
    signal.signal(signal.SIGTERM, signal_handler)
    signal.signal(signal.SIGINT, signal_handler)
```

> apps.py

```
from pathlib import Path
from flask import Blueprint, Flask

__all__ = ["app"]

app = Flask(__name__)

pages_dir = [
    Path(__file__).parent,
    Path(__file__).parent.parent / "api" / "apps",
    Path(__file__).parent.parent / "api" / "apps" / "sdk",
]

client_urls_prefix = [
    register_page(path) for dir in pages_dir for path in search_pages_path(dir)
]
```

> 包安装

```
pip install gunicorn==23.0.0 gevent==24.2.1
```

> entrypoint.sh

```
host=0.0.0.0
port=9380

while [ 1 -eq 1 ];do
    python3 -m gunicorn -w 2 -k gevent -b $host:$port -c /api/gunicorn_config.py api.apps:app
done

wait;
```

**进程生命周期钩子**

| 钩子函数                    | 触发时机                  | 作用                                                         |
| --------------------------- | ------------------------- | ------------------------------------------------------------ |
| `on_starting(server)`       | **主进程启动时**          | 在 Gunicorn **启动之前** 执行，适合做全局初始化              |
| `post_fork(server, worker)` | **Worker 进程 fork 之后** | 适合 Worker 进程的初始化操作                                 |
| `pre_fork(server, worker)`  | **Worker 进程 fork 之前** | 在主进程里执行，适用于日志初始化、环境变量设置               |
| `pre_exec(server)`          | **执行 exec 之前**        | 在 Gunicorn 重新执行（restart）前触发                        |
| `when_ready(server)`        | **主进程完成启动**        | 适用于 Gunicorn **完成所有 worker 进程初始化后**，可以用来发送通知 |
| `on_exit(server)`           | **主进程退出前**          | 适合做清理工作，如释放资源、发送日志等进程生命周期钩子       |

**Worker 进程相关钩子**

| 钩子函数                                   | 触发时机                     | 作用                                         |
| ------------------------------------------ | ---------------------------- | -------------------------------------------- |
| `pre_request(worker, req)`                 | **收到请求之前**             | 适用于日志记录、限流等操作                   |
| `post_request(worker, req, environ, resp)` | **请求处理完毕**             | 适用于记录访问日志、统计请求数等             |
| `worker_int(worker)`                       | **Worker 进程收到 `SIGINT`** | 适合做 Worker 进程内的清理，如关闭数据库连接 |
| `worker_abort(worker)`                     | **Worker 进程因超时被杀死**  | 可以用于记录异常日志                         |
| `worker_exit(server, worker)`              | **Worker 进程退出**          | 可以用于释放资源，比如关闭数据库连接、缓存等 |
| `worker_ready(server, worker)`             | **Worker 进程初始化完成**    | Worker 完成启动后触发                        |
| `nanny_callback(server, worker)`           | **Worker 进程异常时触发**    | 适用于健康检查、异常恢复                     |

## 14、uv

> `uv` 是一个比 `pip` 更快的 Python 依赖管理工具的命令

```
pip3 config set global.index-url https://mirrors.aliyun.com/pypi/simple && \
pip3 config set global.trusted-host mirrors.aliyun.com; \
mkdir -p /etc/uv && \
echo "[[index]]" > /etc/uv/uv.toml && \
echo 'url = "https://mirrors.aliyun.com/pypi/simple"' >> /etc/uv/uv.toml && \
echo "default = true" >> /etc/uv/uv.toml; \
pipx install uv
```

**pyproject.toml**

```
uv init
```

> `pyproject.toml` 取代了 `setup.py` 和 `requirements.txt`，让 Python 项目更加标准化。

```
uv add requests
```

**uv.lock**

>  重新解析 `pyproject.toml` 并更新 `uv.lock`

```
uv resolve
```

> 遇到依赖冲突，可以删除 uv.lock 并重新解析：

```
rm uv.lock
uv resolve
```

> 用于**在 Python 3.10 环境下，根据 `uv.lock` 严格安装锁定版本的依赖**，不会重新解析 `pyproject.toml`

```
uv sync --python 3.10 --frozen
```

> `--frozen`：严格按照锁定文件（如 `requirements.txt` 或 `poetry.lock` / `pipenv.lock` / `uv.lock`）中的版本安装，**不做版本解析**（类似于 `pip install --require-hashes`）。

| 命令                                               | 速度 | 依赖解析 | 严格版本         |
| -------------------------------------------------- | ---- | -------- | ---------------- |
| `pip install -r requirements.txt`                  | 慢   | 需要解析 | 允许部分版本变动 |
| `pip install --require-hashes -r requirements.txt` | 慢   | 需要解析 | 严格             |
| `uv pip install -r requirements.txt`               | 快   | 需要解析 | 允许部分版本变动 |
| `uv sync --frozen`                                 | 快   | 不解析   | 严格             |

| 命令                        | 描述                                    |
| --------------------------- | --------------------------------------- |
| `run`                       | 运行命令或脚本                          |
| `init`                      | 创建一个新项目                          |
| `add`                       | 向项目中添加依赖项                      |
| `remove`                    | 从项目中移除依赖项                      |
| `sync`                      | 更新项目的环境                          |
| `lock`                      | 更新项目的锁定文件                      |
| `export`                    | 将项目的锁定文件导出为其他格式          |
| `tree`                      | 显示项目的依赖树                        |
| `tool`                      | 运行和安装由 Python 包提供的命令        |
| `python`                    | 管理 Python 版本和安装                  |
| `pip`                       | 使用兼容 pip 的接口管理 Python 包       |
| `venv`                      | 创建虚拟环境                            |
| `build`                     | 将 Python 包构建为源代码分发包和 wheels |
| `publish`                   | 将分发包上传到索引                      |
| `cache`                     | 管理 uv 的缓存                          |
| `self`                      | 管理 uv 可执行文件                      |
| `version`                   | 显示 uv 的版本                          |
| `generate-shell-completion` | 生成 shell 自动补全脚本                 |
| `help`                      | 显示某个命令的文档                      |

| 执行命令        | 环境处理                                                     |
| --------------- | ------------------------------------------------------------ |
| `uv run xxx`    | **自动关联虚拟环境： - 优先使用当前目录下的`.venv` - 若不存在会自动创建 - 无需手动激活/停用** |
| `python xxx.py` | **依赖当前Shell环境： - 需手动激活虚拟环境**                 |

sync

```
# 同步所有依赖（包括dev）
uv sync

# 仅同步生产依赖
uv sync --production

# 同步并清理多余包
uv sync --clean
```

lock

```
# 生成新锁定文件
uv lock

# 检查更新但不写入（dry-run）
uv lock --check

# 强制重新解析
uv lock --update
```

tree

```
# 显示完整依赖树
uv tree

# 仅显示指定包的依赖路径
uv tree flask

# 反向追溯依赖（谁依赖了这个包）
uv tree --reverse sqlalchemy

# 输出为JSON格式
uv tree --format json
```

uv python

| 命令        | 描述                     |
| ----------- | ------------------------ |
| `list`      | 列出可用的Python安装版本 |
| `install`   | 下载并安装Python版本     |
| `find`      | 显示当前Python安装位置   |
| `pin`       | 固定使用特定Python版本   |
| `dir`       | 显示uv Python安装目录    |
| `uninstall` | 卸载Python版本           |

## 15、skywalking

```
uv pip install apache-skywalking==1.1.0
```

```
def register_skywalking_server():
    services = os.environ.get('SW_AGENT_COLLECTOR_BACKEND_SERVICES')
    if services:
        logging.info("starting registered to skywalking..")
        from skywalking import agent
        try:
            agent.start()   
            logging.info("Successfully register to skywalking")
        except Exception as e:
            logging.error("Failed to register to skywalking: %s", e)
```

## 16、logging

```
logging.getLogger("nacos").setLevel(logging.WARNING)

logging.getLogger("nacos").disabled = True

logging.getLogger("nacos.heartbeat").setLevel(logging.WARNING)
```

