# pandas 相关

## 一、安装

```
pip install pandas matplotlib -i https://pypi.tuna.tsinghua.edu.cn/simple
```

> #ImportError: No module named Tkinter

```
yum install -y tkinter tk-devel
apt install python-tk 
```

## 二、基本使用

> Series方法与DataFrame差不多，这里只介绍后者如何使用，前者相似。

```
df = pd.DataFrame(np.arange(12).reshape(3,4),columns=['A', 'B', 'C', 'D'])

In [4]: df
Out[4]:
   A  B   C   D
0  0  1   2   3
1  4  5   6   7
2  8  9  10  11
axis=1（按列方向操作）、inplace=True（修改完数据，在原数据上保存）
```

##### 1、按标签来删除列

```
df.drop(['B','C'],axis=1,inplace=True)

   A   D
0  0   3
1  4   7
2  8  11
```

##### 2、按序号来删除列

```
x = [1,2]    #删除多列需给定列表，否则参数过多

df.drop(df.columns[x],axis=1,inplace=True)

   A   D
0  0   3
1  4   7
2  8  11
```

##### 3、按序号来删除行

```
df.drop([0,1],inplace=True)        #默认axis=0

   A  B   C   D
2  8  9  10  11
```

[使用Pandas&NumPy进行数据清洗的6大常用方法](http://bigdata.51cto.com/art/201804/569690.htm)

```
import pandas as pd
import numpy as np

df = pd.DataFrame({"id":[1001,1002,1003,1004,1005,1006], 
 "date":pd.date_range('20130102', periods=6),
  "city":['Beijing ', 'SH', ' guangzhou ', 'Shenzhen', 'shanghai', 'BEIJING '],
 "age":[23,44,54,32,34,32],
 "category":['100-A','100-B','110-A','110-C','210-A','130-F'],
  "price":[1200,np.nan,2133,5433,np.nan,4432]},
  columns =['id','date','city','category','age','price'])
```

##### 4、数据表信息查看 #####

> 查看维度

```
df.shape
```

> 数据表基本信息（维度、列名称、数据格式、所占空间等）

```
df.info()
```

> 每一列数据的格式

```
df.dtypes
```

> 某一列格式

```
df['id'].dtype
```

> 空值

```
df.isnull()
```

> 查看某一列的唯一值

```
df['id'].unique()
```

> 查看数据表的值

```
df.values 
```

> 查看列名称

```
df.columns
```

> 查看前10行数据、后10行数据

```
df.head() #默认前10行数据
df.tail()    #默认后10 行数据
```

##### 5、数据表清洗 #####

> 用数字0填充空值

```
df.fillna(value=0)
```

> 使用列prince的均值对NA进行填充

```
df['prince'].fillna(df['prince'].mean())
```

> 清楚city字段的字符空格

```
df['city']=df['city'].map(str.strip)
```

> 大小写转换

```
df['city']=df['city'].str.lower()
```

> 更改数据格式

```
df['price'].astype('int')    
```

> 更改列名称

```
df.rename(columns={'category': 'category-size'}) 
```

> 删除后出现的重复值

```
df['city'].drop_duplicates()
```

> 删除先出现的重复值

```
df['city'].drop_duplicates(keep='last')
```

> 数据替换

```
df['city'].replace('sh', 'shanghai')
```

> 返回的是个true或false的Series对象（掩码对象），进而筛选出我们需要的特定数据。

```
df[df.isnull()]  

df[df.notnull()]
```

> 将所有含有nan项的row删除

```
df.dropna()
```

> 将在列的方向上三个为NaN的项删除

```
df.dropna(axis=1,thresh=3)
```

> 将全部项都是nan的row删除

```
df.dropna(how='ALL')
```

> 此处：print data.dropna() 和 print data[data.notnull()] 结果一样

> 填充无效值

```
df.fillna(0)
```

> 对第一列nan值赋0，第二列赋值0.5

```
df.fillna({1:0, 2:0.5})
```

> 在列方向上以前一个值作为值赋给NaN

```
df.fillna(method='ffill')
```

> 删除行

```
print frame.drop(['a'])
```

> 删除列，drop函数默认删除行，列需要加axis = 1

```
print frame.drop(['Ohio'], axis = 1)
```

##### 6、数据预处理 #####

```
df1=pd.DataFrame({"id":[1001,1002,1003,1004,1005,1006,1007,1008], 
"gender":['male','female','male','female','male','female','male','female'],
"pay":['Y','N','Y','Y','N','Y','N','Y',],
"m-point":[10,12,20,40,40,40,30,20]})
```

> 数据表合并

```
df_inner=pd.merge(df,df1,how='inner')  # 匹配合并，交集
df_left=pd.merge(df,df1,how='left')        #
df_right=pd.merge(df,df1,how='right')
df_outer=pd.merge(df,df1,how='outer')  #并集
```

> 设置索引列

```
df_inner.set_index('id')
```

> 按照特定列的值排序

```
df_inner.sort_values(by=['age'])
```

> 按照索引列排序

```
df_inner.sort_index()
```

> 如果prince列的值>3000，group列显示high，否则显示low

```
df_inner['group'] = np.where(df_inner['price'] > 3000,'high','low')
```

> 对复合多个条件的数据进行分组标记

```
df_inner.loc[(df_inner['city'] == 'beijing') & (df_inner['price'] >= 4000), 'sign']=1
```

> 对category字段的值依次进行分列，并创建数据表，索引值为df_inner的索引列，列名称为category和size

```
pd.DataFrame((x.split('-') for x in df_inner['category']),index=df_inner.index,columns=['category','size']))
```

> 将完成分裂后的数据表和原df_inner数据表进行匹配

```
df_inner=pd.merge(df_inner,split,right_index=True, left_index=True)
```

##### 7、数据提取 #####

> 主要用到的三个函数：loc,iloc和ix，loc函数按标签值进行提取，iloc按位置进行提取，ix可以同时按标签和位置进行提取。 

> 按索引提取单行的数值

```
df_inner.loc[3]
```

> 按索引提取区域行数值

```
df_inner.iloc[0:5]
```

> 重设索引

```
df_inner.reset_index()
```

> 设置日期为索引

```
df_inner=df_inner.set_index('date') 
```

> 提取4日之前的所有数据

```
df_inner[:'2013-01-04']
```

> 使用iloc按位置区域提取数据

```
df_inner.iloc[:3,:2] #冒号前后的数字不再是索引的标签名称，而是数据所在的位置，从0开始，前三行，前两列。
```

> 适应iloc按位置单独提起数据

```
df_inner.iloc[[0,2,5],[4,5]] #提取第0、2、5行，4、5列
```

> 使用ix按索引标签和位置混合提取数据

```
df_inner.ix[:'2013-01-03',:4] #2013-01-03号之前，前四列数据
```

> 判断city列的值是否为北京

```
df_inner['city'].isin(['beijing'])
```

> 判断city列里是否包含beijing和shanghai，然后将符合条件的数据提取出来

```
df_inner.loc[df_inner['city'].isin(['beijing','shanghai'])] 
```

> 提取前三个字符，并生成数据表

```
pd.DataFrame(category.str[:3])
```

##### 8、数据筛选 #####

> 使用与、或、非三个条件配合大于、小于、等于对数据进行筛选，并进行计数和求和。 

> 使用“与”进行筛选

```
df_inner.loc[(df_inner['age'] > 25) & (df_inner['city'] == 'beijing'), ['id','city','age','category','gender']]
```

> 使用“或”进行筛选

```
df_inner.loc[(df_inner['age'] > 25) | (df_inner['city'] == 'beijing'), ['id','city','age','category','gender']].sort(['age']) 
```

> 使用“非”条件进行筛选

```
df_inner.loc[(df_inner['city'] != 'beijing'), ['id','city','age','category','gender']].sort(['id']) 
```

> 对筛选后的数据按city列进行计数

```
df_inner.loc[(df_inner['city'] != 'beijing'), ['id','city','age','category','gender']].sort(['id']).city.count()
```

> 使用query函数进行筛选

```
df_inner.query('city == ["beijing", "shanghai"]')
```

> 对筛选后的结果按prince进行求和

```
df_inner.query('city == ["beijing", "shanghai"]').price.sum()
```

##### 9、数据汇总 ##### 
> 主要函数是groupby和pivote_table 

> 对所有的列进行计数汇总

```
df_inner.groupby('city').count()
```

> 按城市对id字段进行计数

```
df_inner.groupby('city')['id'].count()
```

> 对两个字段进行汇总计数

```
df_inner.groupby(['city','size'])['id'].count()
```

> 对city字段进行汇总，并分别计算prince的合计和均值

```
df_inner.groupby('city')['price'].agg([len,np.sum, np.mean]) 
```

##### 10、数据统计 ##### 
> 数据采样，计算标准差，协方差和相关系数 

> 简单的数据采样

```
df_inner.sample(n=3) 
```

> 手动设置采样权重

```
weights = [0, 0, 0, 0, 0.5, 0.5]
df_inner.sample(n=2, weights=weights) 
```

> 采样后不放回

```
df_inner.sample(n=6, replace=False) 
```

> 采样后放回

```
df_inner.sample(n=6, replace=True)
```

> 数据表描述性统计

```
df_inner.describe().round(2).T #round函数设置显示小数位，T表示转置
```

> 计算列的标准差

```
df_inner['price'].std()
```

> 计算两个字段间的协方差

```
df_inner['price'].cov(df_inner['m-point']) 
```

> 数据表中所有字段间的协方差

```
df_inner.cov()
```

> 两个字段的相关性分析

```
df_inner['price'].corr(df_inner['m-point']) #相关系数在-1到1之间，接近1为正相关，接近-1为负相关，0为不相关
```

> 数据表的相关性分析

```
df_inner.corr()
```

##### 11、数据输出 ##### 
> 分析后的数据可以输出为xlsx格式和csv格式 

> 写入Excel

```
df_inner.to_excel('excel_to_python.xlsx', sheet_name='bluewhale_cc') 
```

> 写入到CSV

```
df_inner.to_csv('excel_to_python.csv') 
```

## 三、实例

```
# -*- coding: UTF-8 -*-

import pandas as pd

name = []
score = []

with open('film_log3.csv','r') as film_data:
  for i in film_data:
    name.append(i.split(';')[0])
    score.append(i.split(';')[7])

#print name[0]
movie = pd.DataFrame({'nameinfo':name,'scoreinfo':score})

print movie.head()
movie = pd.read_csv('film_log3.csv',prefix='tmp',header=None)

#print movie
movie_split = pd.DataFrame((x.split(';') for x in movie.tmp0),index=movie.index,columns=['name','2','3','4','5','6','7','score','9'])

movie = movie_split.ix[:,[0,7]]

#movie.to_csv('out.csv')

#print movie[0:1]
#[df['列名'].isin([相应的值])]

print movie
```

