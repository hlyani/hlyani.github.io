# Python Sqlalchemy

## 一、安装

```
pip install sqlalchemy
```

##### 1、安装mysql

```
apt-get install mysql-server
apt-get install mysql-client
apt-get install libmysqlclient15-dev
```

##### 2、安装python-mysqldb

```
apt-get install python-mysqldb
```

##### 3、easy_install

```
wget http://peak.telecommunity.com/dist/ez_setup.py
python ez_setup.py
```

##### 4、MySQL-python

```
easy_install MySQL-Python
```

##### 5、SQLAlchemy

```
easy_install SQLAlchemy
```

##### 6、导包测试

```
vim test.py

from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker
```

> DB_CONNECT_STRING 连接数据库的路径，mysql+mysqldb 指定使用MySQL-Python来连接，“ooxx”是使用的数据库名（可省略），“charset”指定了连接时使用的字符集（可省略）。

```
DB_CONNECT_STRING = 'mysql+mysqldb://root:123@localhost/ooxx?charset=utf8'
```

> create_engine() 会返回一个数据库引擎，echo 参数为 True 时，会显示每条执行的 SQL 语句，生产环境下可关闭。

```
engine = create_engine(DB_CONNECT_STRING, echo=True)
```

> sessionmaker() 会生成一个数据库会话类。这个类的实例可以当成一个数据库连接，它同时还记录了一些查询的数据，并决定什么时候执行 SQL 语句。由于 SQLAlchemy 自己维护了一个数据库连接池（默认 5 个连接），因此初始化一个会话的开销并不大。对 Tornado 而言，可以在 BaseHandler 的 initialize() 里初始化：

```
DB_Session = sessionmaker(bind=engine)
session = DB_Session()
```

代码如下:

```
session.execute('create database abc')
print session.execute('show databases').fetchall()
session.execute('use abc')
建 user 表的过程略

print session.execute('select * from user where id = 1').first()
print session.execute('select * from user where id = :id', {'id': 1}).first()
```

##### 定义一个表：

```
from sqlalchemy import Column
from sqlalchemy.types import CHAR, Integer, String
from sqlalchemy.ext.declarative import declarative_base

BaseModel = declarative_base()
def init_db():
    BaseModel.metadata.create_all(engine)
def drop_db():
    BaseModel.metadata.drop_all(engine)

class User(BaseModel):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    name = Column(CHAR(30)) # or Column(String(30))
init_db()
```

```
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column,Integer,String
from sqlalchemy.orm import sessionmaker
from sqlalchemy import create_engine

# DB_CONNECT_STRING = 'mysql_mysqldb://root:0127@localhost/ooxx?charset=utf8'

DB_CONNECT_STRING = 'mysql+mysqldb://root:0127@172.18.50.7/test'
engine = create_engine(DB_CONNECT_STRING,echo=True)
DB_Session = sessionmaker(bind=engine)
session = DB_Session()

Base = declarative_base()

# BaseModel.metadata.create_all(engine) 会找到 BaseModel 的所有子类，并在数据库中建立这些表

class User(Base):

# __tablename__ 属性就是数据库中该表的名称

  __tablename__ = 'user'
  id = Column(Integer,primary_key=True)
  name = Column(String)
  fullname = Column(String(30))
  password = Column(String(30))

# name = Column(CHAR(30)) 

ed_user = User(name='ll',fullname='lil',password='0829')

session.add(ed_user)
session.commit()

query = session.query(User)
```

##### 显示SQL 语句
```
print(query)
```

##### 显示SQL 语句
```
print(query.statement)
```

##### 遍历时查询
```
for user in query:
  print(user.name)
```

 ##### 返回的是一个类似列表的对象
```
print(query.all())

print(query.first().name) # 记录不存在时，first() 会返回 None
```

```
# print(query.one().name) # 不存在，或有多行记录时会抛出异常

print query.filter(User.id == 2).first().name
print query.get(2).name # 以主键获取，等效于上句
print query.filter('id = 2').first().name # 支持字符串
query2 = session.query(User.name)
print query2.all() # 每行是个元组
print query2.limit(1).all() # 最多返回 1 条记录
print query2.offset(1).all() # 从第 2 条记录开始返回
print query2.order_by(User.name).all()
print query2.order_by('name').all()
print query2.order_by(User.name.desc()).all()
print query2.order_by('name desc').all()
print session.query(User.id).order_by(User.name.desc(), User.id).all()
print query2.filter(User.id == 1).scalar() # 如果有记录，返回第一条记录的第一个元素
print session.query('id').select_from(User).filter('id = 1').scalar()
print query2.filter(User.id > 1, User.name != 'a').scalar() # and
query3 = query2.filter(User.id > 1) # 多次拼接的 filter 也是 and
query3 = query3.filter(User.name != 'a')
print query3.scalar()
print query2.filter(or_(User.id == 1, User.id == 2)).all() # or
print query2.filter(User.id.in_((1, 2))).all() # in
query4 = session.query(User.id)
print query4.filter(User.name == None).scalar()
print query4.filter('name is null').scalar()
print query4.filter(not_(User.name == None)).all() # not
print query4.filter(User.name != None).all()
print query4.count()
print session.query(func.count('*')).select_from(User).scalar()
print session.query(func.count('1')).select_from(User).scalar()
print session.query(func.count(User.id)).scalar()
print session.query(func.count('*')).filter(User.id > 0).scalar() # filter() 中包含 User，因此不需要指定表
print session.query(func.count('*')).filter(User.name == 'a').limit(1).scalar() == 1 # 可以用 limit() 限制 count() 的返回数
print session.query(func.sum(User.id)).scalar()
print session.query(func.now()).scalar() # func 后可以跟任意函数名，只要该数据库支持
print session.query(func.current_timestamp()).scalar()
print session.query(func.md5(User.name)).filter(User.id == 1).scalar()
query.filter(User.id == 1).update({User.name: 'c'})
user = query.get(1)
print user.name
user.name = 'd'
session.flush() # 写数据库，但并不提交
print query.get(1).name
session.delete(user)
session.flush()
print query.get(1)
session.rollback()
print query.get(1).name
query.filter(User.id == 1).delete()
session.commit()
print query.get(1)

通过Session的query()方法创建一个查询对象。这个函数的参数数量是可变的，参数可以是任何类或者是类的描述的集合。下面是一个迭代输出User类的例子：

for instance in session.query(User).order_by(User.id):
  print instance.name,instance.fullname

Query也支持ORM描述作为参数。任何时候，多个类的实体或者是基于列的实体表达都可以作为query()函数的参数，返回类型是元组：

for name, fullname in session.query(User.name,User.fullname): 
  print name, fullname

使用关键字变量过滤查询结果，filter 和 filter_by都适用。【2】使用很简单，下面列出几个常用的操作：

query.filter(User.name == 'ed') #equals
query.filter(User.name != 'ed') #not equals
query.filter(User.name.like('%ed%')) #LIKE
uery.filter(User.name.in_(['ed','wendy', 'jack'])) #IN
query.filter(User.name.in_(session.query(User.name).filter(User.name.like('%ed%'))#IN
query.filter(~User.name.in_(['ed','wendy', 'jack']))#not IN
query.filter(User.name == None)#is None
query.filter(User.name != None)#not None
from sqlalchemy import and_
query.filter(and_(User.name =='ed',User.fullname =='Ed Jones')) # and
query.filter(User.name == 'ed',User.fullname =='Ed Jones') # and
query.filter(User.name == 'ed').filter(User.fullname == 'Ed Jones')# and
from sqlalchemy import or_
query.filter(or_(User.name =='ed', User.name =='wendy')) #or
query.filter(User.name.match('wendy')) #match
```

