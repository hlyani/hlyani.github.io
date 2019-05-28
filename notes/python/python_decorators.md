# Python 装饰器
> 装饰器本质上是一个Python函数，它可以让其他函数在不需要做任何代码变动的前提下增加额外功能，装饰器的返回值也是一个函数对象。它经常用于有切面需求的场景，比如：插入日志、性能测试、事务处理、缓存、权限校验等场景。装饰器是解决这类问题的绝佳设计，有了装饰器，我们就可以抽离出大量与函数功能本身无关的雷同代码并继续重用。

<https://www.cnblogs.com/cicaday/p/python-decorator.html>

##### 1、python 2.4之前实现，函数添加额外功能
```
def debug(func):
    def wrapper():
        print "[DEBUG]: enter {}()".format(func.__name__)
        return func()
    return wrapper

def say_hello():
    print "hello!"

say_hello = debug(say_hello)
```
##### 2、支持@语法糖后
```
def debug(func):
    def wrapper():
        print "[DEBUG]: enter {}()".format(func.__name__)
        return func()
    return wrapper

@debug
def say_hello():
    print "hello!"
```
##### 3、被装饰函数传入参数
```
def debug(func):
    def wrapper(something):  # 指定一毛一样的参数
        print "[DEBUG]: enter {}()".format(func.__name__)
        return func(something)
    return wrapper  # 返回包装过函数

@debug
def say(something):
    print "hello {}!".format(something)
```
##### 4、被装饰函数传入可变参数
```
def debug(func):
    def wrapper(*args, **kwargs):  # 指定宇宙无敌参数
        print "[DEBUG]: enter {}()".format(func.__name__)
        print 'Prepare and say...',
        return func(*args, **kwargs)
    return wrapper  # 返回

@debug
def say(something):
    print "hello {}!".format(something)
```
##### 5、装饰器传入参数
```
def logging(level):
    def wrapper(func):
        def inner_wrapper(*args, **kwargs):
            print "[{level}]: enter function {func}()".format(
                level=level,
                func=func.__name__)
            return func(*args, **kwargs)
        return inner_wrapper
    return wrapper

@logging(level='INFO')
def say(something):
    print "say {}!".format(something)

# 如果没有使用@语法，等同于
# say = logging(level='INFO')(say)

@logging(level='DEBUG')
def do(something):
    print "do {}...".format(something)

if __name__ == '__main__':
    say('hello')
    do("my work")
```
##### 6、基于类实现的装饰器，让类的构造函数__init__()接受一个函数，然后重载__call__()并返回一个函数，也可以达到装饰器函数的效果。
```
class logging(object):
    def __init__(self, func):
        self.func = func

    def __call__(self, *args, **kwargs):
        print "[DEBUG]: enter function {func}()".format(
            func=self.func.__name__)
        return self.func(*args, **kwargs)
@logging
def say(something):
    print "say {}!".format(something)
```
##### 7、带参数的类装饰器
```
class logging(object):
    def __init__(self, level='INFO'):
        self.level = level
        
    def __call__(self, func): # 接受函数
        def wrapper(*args, **kwargs):
            print "[{level}]: enter function {func}()".format(
                level=self.level,
                func=func.__name__)
            func(*args, **kwargs)
        return wrapper  #返回函数

@logging(level='INFO')
def say(something):
    print "say {}!".format(something)
```
##### 8、装饰器 @property
@property
```
def getx(self):
    return self._x

def setx(self, value):
    self._x = value
    
def delx(self):
   del self._x
   
x = property(getx, setx, delx, "I am doc for x property")

@property
def x(self): ...

# 等同于
def x(self): ...
x = property(x)

# 例子

def __init__(self):
	self._role_id = None

@property    
def role_id(self):
	if not self._role_id:
		self._role_id = self.keystone.roles.find(name = '_member_').id
	return self._role_id

def _grant_role(self,openstack_user):
	self.keystone.roles.grant(
		self.role_id, 
		user = openstack_user.id, 
		project = self.default_project)
```
##### 9、wraps 作用
```
# wraps作用 https://blog.csdn.net/hqzxsc2006/article/details/50337865
Python装饰器（decorator）在实现的时候，被装饰后的函数其实已经是另外一个函数了（函数名等函数属性会发生改变），为了不影响，Python的functools包中提供了一个叫wraps的decorator来消除这样的副作用。写一个decorator的时候，最好在实现之前加上functools的wrap，它能保留原有函数的名称和docstring。

from functools import wraps   
def my_decorator(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        '''decorator'''
        print('Calling decorated function...')
        return func(*args, **kwargs)
    return wrapper  

@my_decorator 
def example():
    """Docstring""" 
    print('Called example function')
print(example.__name__, example.__doc__)

# 不使用wraps结果
('wrapper', 'decorator')

# 使用wraps结果
('example', 'Docstring')

# 例子
from functools import wraps
 
def logit(func):
    @wraps(func)
    def with_logging(*args, **kwargs):
        print(func.__name__ + " was called")
        return func(*args, **kwargs)
    return with_logging
 
@logit
def addition_func(x):
   """Do some math."""
   return x + x
 
result = addition_func(4)
```
##### 10、装饰器 @staticmethod
```
返回函数的静态方法,无需实例化就可以使用。
 
class C(object):
    @staticmethod
    def f():
        print('runoob');
 
C.f();          # 静态方法无需实例化
cobj = C()
cobj.f()        # 也可以实例化后调用
```
##### 11、装饰器 @classmethod
```
classmethod 修饰符对应的函数不需要实例化，不需要 self 参数，但第一个参数需要是表示自身类的 cls 参数，可以来调用类的属性，类的方法，实例化对象等。

class A(object):
    bar = 1
    def func1(self):  
        print ('foo') 
    @classmethod
    def func2(cls):
        print ('func2')
        print (cls.bar)
        cls().func1()   # 调用 foo 方法
 
A.func2()               # 不需要实例化
```
##### 12、闭包
> 在一些语言中，在函数中可以（嵌套）定义另一个函数时，如果内部的函数引用了外部的函数的变量，则可能产生闭包。闭包可以用来在一个函数与一组“私有”变量之间创建关联关系。在给定函数被多次调用的过程中，这些私有变量能够保持其持久性。

> 用比较容易懂的人话说，就是当某个函数被当成对象返回时，夹带了外部变量，就形成了一个闭包。

```
def make_printer(msg):
    def printer():
        print msg  # 夹带私货（外部变量）
    return printer  # 返回的是函数，带私货的函数

printer = make_printer('Foo!')
printer()
```
<https://www.cnblogs.com/cicaday/p/python-decorator.html>