# Python logging模块

## 一、介绍

##### 1、日志级别

> 日志一共分成5个等级，从低到高分别是：NOTSET < DEBUG < INFO < WARNING < ERROR < CRITICAL。

* DEBUG：详细的信息，通常只出现在诊断问题上
* INFO：确认一切按预期运行
* WARNING：一个迹象表明，一些意想不到的事情发生了，或表明一些问题在不久的将来(例如：磁盘空间低)。这个软件还能按预期工作。
* ERROR：更严重的问题，软件没能执行一些功能
* CRITICAL：一个严重的错误,这表明程序本身可能无法继续运行

> 这5个等级，也分别对应5种打日志的方法： debug 、info 、warning 、error 、critical。默认的是WARNING，当在WARNING或之上时才被跟踪。

> 既要把日志输出到控制台， 还要写入日志文件
> 这就需要一个叫作Logger 的对象来帮忙，下面将对他进行详细介绍，现在这里先学习怎么实现把日志既要输出到控制台又要输出到文件的功能。

## 二、使用

##### 1、第一步，创建一个logger  
```
import logging  

logger = logging.getLogger()  
logger.setLevel(logging.INFO)    # Log等级总开关  
```

##### 2、第二步，创建一个handler，用于写入日志文件  
```
logfile = './log/logger.txt'  
fh = logging.FileHandler(logfile, mode='w')  
fh.setLevel(logging.DEBUG)   # 输出到file的log等级的开关  
```

##### 3、第三步，再创建一个handler，用于输出到控制台  
```
ch = logging.StreamHandler()  
ch.setLevel(logging.WARNING)   # 输出到console的log等级的开关  
```

##### 4、第四步，定义handler的输出格式  
```
formatter = logging.Formatter("%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s")  
fh.setFormatter(formatter)  
ch.setFormatter(formatter)  
```

##### 5、第五步，将logger添加到handler里面  
```
logger.addHandler(fh)  
logger.addHandler(ch)]()  
```

## 三、日志
```
logger.debug('this is a logger debug message')  
logger.info('this is a logger info message')  
logger.warning('this is a logger warning message')  
logger.error('this is a logger error message')  
logger.critical('this is a logger critical message')  

日志格式说明
logging.basicConfig函数中，可以指定日志的输出格式format，这个参数可以输出很多有用的信息，如下所示：

%(levelno)s: 打印日志级别的数值
%(levelname)s: 打印日志级别名称
%(pathname)s: 打印当前执行程序的路径，其实就是sys.argv[0]
%(filename)s: 打印当前执行程序名
%(funcName)s: 打印日志的当前函数
%(lineno)d: 打印日志的当前行号
%(asctime)s: 打印日志的时间
%(thread)d: 打印线程ID
%(threadName)s: 打印线程名称
%(process)d: 打印进程ID
%(message)s: 打印日志信息

我在工作中给的常用格式在前面已经看到了。就是：
format='%(asctime)s - %(filename)s[line:%(lineno)d] - %(levelname)s: %(message)s'
```

##### 1、简单的将日志打印到屏幕

```
import logging

logging.debug('This is debug message')
logging.info('This is info message')
logging.warning('This is warning message')

屏幕上打印:
WARNING:root:This is warning message
```

##### 2、通过logging.basicConfig函数对日志的输出格式及方式做相关配置

```
import logging

logging.basicConfig(level=logging.DEBUG,
                format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
                datefmt='%a, %d %b %Y %H:%M:%S',
                filename='myapp.log',
                filemode='w')
    
logging.debug('This is debug message')
logging.info('This is info message')
logging.warning('This is warning message')

./myapp.log文件中内容为:
Sun, 24 May 2009 21:48:54 demo2.py[line:11] DEBUG This is debug message
Sun, 24 May 2009 21:48:54 demo2.py[line:12] INFO This is info message
Sun, 24 May 2009 21:48:54 demo2.py[line:13] WARNING This is warning message

logging.basicConfig函数各参数:
filename: 指定日志文件名
filemode: 和file函数意义相同，指定日志文件的打开模式，'w'或'a'
format: 指定输出的格式和内容，format可以输出很多有用信息，如上例所示:
 %(levelno)s: 打印日志级别的数值
 %(levelname)s: 打印日志级别名称
 %(pathname)s: 打印当前执行程序的路径，其实就是sys.argv[0]
 %(filename)s: 打印当前执行程序名
 %(funcName)s: 打印日志的当前函数
 %(lineno)d: 打印日志的当前行号
 %(asctime)s: 打印日志的时间
 %(thread)d: 打印线程ID
 %(threadName)s: 打印线程名称
 %(process)d: 打印进程ID
 %(message)s: 打印日志信息
datefmt: 指定时间格式，同time.strftime()
level: 设置日志级别，默认为logging.WARNING
stream: 指定将日志的输出流，可以指定输出到sys.stderr,sys.stdout或者文件，默认输出到sys.stderr，当stream和filename同时指定时，stream被忽略
```

##### 3、将日志同时输出到文件和屏幕

```
import logging

logging.basicConfig(level=logging.DEBUG,
                format='%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s',
                datefmt='%a, %d %b %Y %H:%M:%S',
                filename='myapp.log',
                filemode='w')
```

```
定义一个StreamHandler，将INFO级别或更高的日志信息打印到标准错误，并将其添加到当前的日志处理对象

console = logging.StreamHandler()
console.setLevel(logging.INFO)
formatter = logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
console.setFormatter(formatter)
logging.getLogger('').addHandler(console)

logging.debug('This is debug message')
logging.info('This is info message')
logging.warning('This is warning message')

屏幕上打印:
root        : INFO     This is info message
root        : WARNING  This is warning message

./myapp.log文件中内容为:
Sun, 24 May 2009 21:48:54 demo2.py[line:11] DEBUG This is debug message
Sun, 24 May 2009 21:48:54 demo2.py[line:12] INFO This is info message
Sun, 24 May 2009 21:48:54 demo2.py[line:13] WARNING This is warning message
```

##### 4、logging之日志回滚

```
import logging
from logging.handlers import RotatingFileHandler

定义一个RotatingFileHandler，最多备份5个日志文件，每个日志文件最大10M

Rthandler = RotatingFileHandler('myapp.log', maxBytes=10*1024*1024,backupCount=5)
Rthandler.setLevel(logging.INFO)
formatter = logging.Formatter('%(name)-12s: %(levelname)-8s %(message)s')
Rthandler.setFormatter(formatter)
logging.getLogger('').addHandler(Rthandler)

从上例和本例可以看出，logging有一个日志处理的主对象，其它处理方式都是通过addHandler添加进去的。
logging的几种handle方式如下：

logging.StreamHandler: 日志输出到流，可以是sys.stderr、sys.stdout或者文件
logging.FileHandler: 日志输出到文件

日志回滚方式，实际使用时用RotatingFileHandler和TimedRotatingFileHandler
logging.handlers.BaseRotatingHandler
logging.handlers.RotatingFileHandler
logging.handlers.TimedRotatingFileHandler

logging.handlers.SocketHandler: 远程输出日志到TCP/IP sockets
logging.handlers.DatagramHandler:  远程输出日志到UDP sockets
logging.handlers.SMTPHandler:  远程输出日志到邮件地址
logging.handlers.SysLogHandler: 日志输出到syslog
logging.handlers.NTEventLogHandler: 远程输出日志到Windows NT/2000/XP的事件日志
logging.handlers.MemoryHandler: 日志输出到内存中的制定buffer
logging.handlers.HTTPHandler: 通过"GET"或"POST"远程输出到HTTP服务器

由于StreamHandler和FileHandler是常用的日志处理方式，所以直接包含在logging模块中，而其他方式则包含在logging.handlers模块中，
```

##### 5、通过logging.config模块配置日志
```
vim logger.conf

[loggers]
keys=root,example01,example02

[logger_root]
level=DEBUG
handlers=hand01,hand02

[logger_example01]
handlers=hand01,hand02
qualname=example01
propagate=0

[logger_example02]
handlers=hand01,hand03
qualname=example02
propagate=0

[handlers]
keys=hand01,hand02,hand03

[handler_hand01]
class=StreamHandler
level=INFO
formatter=form02
args=(sys.stderr,)

[handler_hand02]
class=FileHandler
level=DEBUG
formatter=form01
args=('myapp.log', 'a')

[handler_hand03]
class=handlers.RotatingFileHandler
level=INFO
formatter=form02
args=('myapp.log', 'a', 10*1024*1024, 5)

[formatters]
keys=form01,form02

[formatter_form01]
format=%(asctime)s %(filename)s[line:%(lineno)d] %(levelname)s %(message)s
datefmt=%a, %d %b %Y %H:%M:%S

[formatter_form02]
format=%(name)-12s: %(levelname)-8s %(message)s
datefmt=

上例3：

import logging
import logging.config

logging.config.fileConfig("logger.conf")
logger = logging.getLogger("example01")

logger.debug('This is debug message')
logger.info('This is info message')
logger.warning('This is warning message')

上例4：

import logging
import logging.config

logging.config.fileConfig("logger.conf")
logger = logging.getLogger("example02")

logger.debug('This is debug message')
logger.info('This is info message')
logger.warning('This is warning message')
```

##### 6、logging是线程安全的

http://blog.csdn.net/yatere/article/details/6655445