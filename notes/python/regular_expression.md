# 正则表达式相关

### 一、常用元字符和语法

| 语法            | 说明                                                         | 实例                       | 匹配结果              |
| --------------- | ------------------------------------------------------------ | -------------------------- | --------------------- |
| .               | 匹配任意除换行符"\n"外的所有字符。                           | a.c                        | abc                   |
| [..]            | 字符集，对应位置可以是字符集中任意字符。                     | a[bcd]e                    | abe<br />ace<br />ade |
| \d              | 数字，[0-9]                                                  | a\dc                       | a1c                   |
| \D              | 非数字，[^\d\]                                               | a\Dc                       | abc                   |
| \s              | 空白字符，匹配任何空白字符，包括空格、制表符、换页符等等，[ \t\r\n\f\v] | a\sc                       | a c                   |
| \S              | 非空白字符，[^\s\]                                           | a\Sc                       | abc                   |
| \w              | 单词字符，[A-Za-z0-9]                                        | a\wc                       | abc                   |
| \W              | 非单词字符，[^\w\]                                           | a\Wc                       | a c                   |
| *               | 匹配前一个字符0次或多次                                      | abc*                       | ab<br />abccc         |
| +               | 匹配前一个字符1次或多次                                      | abc+                       | abc<br />abccc        |
| ?               | 匹配前一个字符0次或1次                                       | abc？                      | ab<br />abc           |
| {m}             | 匹配前一个字符m次                                            | ab{2}c                     | abbc                  |
| {m,n}           | 匹配前一个字符m到n次。<br />m和n可以省略，若省略m，则匹配0到n次；若省略n，则匹配m至多次 | ab{1,2}c                   | abc<br />abbc         |
| ^               | 匹配字符串开头。在多行时，匹配每一行开头。<br />匹配输入字符串的开始位置，除非在方括号表达式中使用，此时它表示不接受该字符集合。 | ^abc                       | abc                   |
| $               | 匹配字符串末尾。在多行时，匹配每一行末尾。                   | abc$                       | abc                   |
| \A              | 仅匹配字符串开头。                                           | \Aabc                      | abc                   |
| \Z              | 仅匹配字符串末尾。                                           | abc\Z                      | abc                   |
| \b              | 匹配\w和\W之间<br />匹配一个单词边界，即字与空格间的位置。   | a\b!bc                     | a!bc                  |
| \B              | 非单词边界匹配。\[^\b\]                                      | a\Bbc                      | abc                   |
| \|              | 左右表达式任意匹配一个                                       | abc\|def                   | abc<br />def          |
| (...)           | 标记一个子表达式的开始和结束位置。子表达式可以获取供以后使用。 | (abc){2}<br />a(123\|456)c | abcabc<br />a456c     |
| [\u4e00-\u9fa5] | 匹配中文字符                                                 |                            |                       |
| [\s\S]*?        | 表示匹配任意字符                                             |                            |                       |

### 二、相关概念

##### 1、数量词的贪婪模式和非贪婪模式

> 正则表达式通常用于在文本中查找匹配的字符串。

* 贪婪模式：总是尝试匹配尽可能多的字符。
* 非贪婪模式：总是尝试匹配尽可能少的字符。

> Python里数量词默认是贪婪的
>
> 例：正则表达式"ab*"，如果用于查找"abbbc"，将找到"abbb"。如果用非贪婪模式的数量词"ab\*?"，将找到"a"。

```
源字符串：aa<div>test1</div>bb<div>test2</div>cc 

正则表达式一：<div>.*</div> # 贪婪模式

匹配结果一：<div>test1</div>bb<div>test2</div> 

正则表达式二：<div>.*?</div> # 非贪婪模式

匹配结果二：<div>test1</div>（这里指的是一次匹配结果，所以没包括<div>test2</div>） 
```

##### 2、原生字符串

> 匹配一个数字的"\\d"可以写成r"\d"

##### 3、flags

-   re.I(全拼：IGNORECASE): 忽略大小写（括号内是完整写法，下同）
-   re.M(全拼：MULTILINE): 多行模式，改变'^'和'$'的行为（参见上图）
-   re.S(全拼：DOTALL): 点任意匹配模式，改变'.'的行为
-   re.L(全拼：LOCALE): 使预定字符类 \w \W \b \B \s \S 取决于当前区域设定
-   re.U(全拼：UNICODE): 使预定字符类 \w \W \b \B \s \S \d \D 取决于unicode定义的字符属性
-   re.X(全拼：VERBOSE): 详细模式。这个模式下正则表达式可以是多行，忽略空白字符，并可以加入注释。

##### 4、group

> 除了简单地判断是否匹配之外，正则表达式还有提取子串的强大功能。用`()`表示的就是要提取的分组（Group）。比如：`^(\d{3})-(\d{3,8})$`分别定义了两个组，可以直接从匹配的字符串中提取出区号和本地号码 

```
m = re.match(r'^(\d{3})-(\d{3,8})$', '010-12345')
print(m.group(0))
print(m.group(1))
print(m.group(2))

# 010-12345
# 010
# 12345
```

### 三、常用方法

##### 1、match

>  re.match 尝试从字符串的起始位置匹配一个模式，如果不是起始位置匹配成功的话，match()就返回none。 

```
print(re.match('com', 'www.test.com'))
```

##### 2、search

>   扫描整个字符串并返回第一个成功的匹配 

```
print(re.search('www', 'www.test.com').span()) 
```

##### 3、findall

>  在字符串中找到正则表达式所匹配的所有子串，并返回一个列表，如果没有找到匹配的，则返回空列表。 

```
import re
 
p = re.compile(r'\d+')
print p.findall('one1two2three3four4')
 
### output ###
# ['1', '2', '3', '4']
```

##### 4、sub

>  sub用于替换字符串中的匹配项 

```
phone = "2004-959-559 # 这是一个电话号码"
 
# 删除注释
num = re.sub(r'#.*$', "", phone)
print ("电话号码 : ", num)
```

##### 5、split

>  split 方法按照能够匹配的子串将字符串分割后返回列表 

```
re.split('\W+', 'runoob, runoob, runoob.')
['runoob', 'runoob', 'runoob', '']
```

##### 6、compile

>  compile 函数用于编译正则表达式，生成一个正则表达式（ Pattern ）对象，供 match() 和 search() 这两个函数使用。 
>
>  如果一个正则表达式要重复使用几千次，出于效率的考虑，我们可以预编译该正则表达式 

```
# 编译
tele = re.compile(r'^(\d{3})-(\d{3,8})$')
# 使用：
print(tele.match('010-12345').groups())
```

##### 7、finditer

>  和 findall 类似，在字符串中找到正则表达式所匹配的所有子串，并把它们作为一个迭代器返回 

```
it = re.finditer(r"\d+","12a32bc43jf3") 
for match in it: 
    print (match.group() )
```