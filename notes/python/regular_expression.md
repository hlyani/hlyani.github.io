# 正则表达式相关

### 一、常用元字符和语法

| 语法  | 说明                                                         | 实例                       | 匹配结果              |
| ----- | ------------------------------------------------------------ | -------------------------- | --------------------- |
| .     | 匹配任意除换行符"\n"外的所有字符。                           | a.c                        | abc                   |
| [..]  | 字符集，对应位置可以是字符集中任意字符。                     | a[bcd]e                    | abe<br />ace<br />ade |
| \d    | 数字，[0-9]                                                  | a\dc                       | a1c                   |
| \D    | 非数字，[^\d\]                                               | a\Dc                       | abc                   |
| \s    | 空白字符，匹配任何空白字符，包括空格、制表符、换页符等等，[ \t\r\n\f\v] | a\sc                       | a c                   |
| \S    | 非空白字符，[^\s\]                                           | a\Sc                       | abc                   |
| \w    | 单词字符，[A-Za-z0-9]                                        | a\wc                       | abc                   |
| \W    | 非单词字符，[^\w\]                                           | a\Wc                       | a c                   |
| *     | 匹配前一个字符0次或多次                                      | abc*                       | ab<br />abccc         |
| +     | 匹配前一个字符1次或多次                                      | abc+                       | abc<br />abccc        |
| ?     | 匹配前一个字符0次或1次                                       | abc？                      | ab<br />abc           |
| {m}   | 匹配前一个字符m次                                            | ab{2}c                     | abbc                  |
| {m,n} | 匹配前一个字符m到n次。<br />m和n可以省略，若省略m，则匹配0到n次；若省略n，则匹配m至多次 | ab{1,2}c                   | abc<br />abbc         |
| ^     | 匹配字符串开头。在多行时，匹配每一行开头。<br />匹配输入字符串的开始位置，除非在方括号表达式中使用，此时它表示不接受该字符集合。 | ^abc                       | abc                   |
| $     | 匹配字符串末尾。在多行时，匹配每一行末尾。                   | abc$                       | abc                   |
| \A    | 仅匹配字符串开头。                                           | \Aabc                      | abc                   |
| \Z    | 仅匹配字符串末尾。                                           | abc\Z                      | abc                   |
| \b    | 匹配\w和\W之间<br />匹配一个单词边界，即字与空格间的位置。   | a\b!bc                     | a!bc                  |
| \B    | 非单词边界匹配。\[^\b\]                                      | a\Bbc                      | abc                   |
| \|    | 左右表达式任意匹配一个                                       | abc\|def                   | abc<br />def          |
| (...) | 标记一个子表达式的开始和结束位置。子表达式可以获取供以后使用。 | (abc){2}<br />a(123\|456)c | abcabc<br />a456c     |

##### 1、数量词的贪婪模式和非贪婪模式

> 正则表达式通常用于在文本中查找匹配的字符串。

* 贪婪模式：总是尝试匹配尽可能多的字符。
* 非贪婪模式：总是尝试匹配尽可能少的字符。

> Python里数量词默认是贪婪的
>
> 例：正则表达式"ab*"，如果用于查找"abbbc"，将找到"abbb"。如果用非贪婪模式的数量词"ab\*?"，将找到"a"。

##### 2、原生字符串

> 匹配一个数字的"\\d"可以写成r"\d"

## 二、re模块

```
-*- coding: utf-8 -*-
#一个简单的re实例，匹配字符串中的hello字符串
import re

m = re.match(r'hello', 'hello world!')
print(m.group())
```

## 1、Compile

* ★ `re.compile(strPattern[, flag]): `

>  比如re.compile('pattern', re.I | re.M)与re.compile('(?im)pattern')是等价的。 

可选值有：

-   re.I(全拼：IGNORECASE): 忽略大小写（括号内是完整写法，下同）
-   re.M(全拼：MULTILINE): 多行模式，改变'^'和'$'的行为（参见上图）
-   re.S(全拼：DOTALL): 点任意匹配模式，改变'.'的行为
-   re.L(全拼：LOCALE): 使预定字符类 \w \W \b \B \s \S 取决于当前区域设定
-   re.U(全拼：UNICODE): 使预定字符类 \w \W \b \B \s \S \d \D 取决于unicode定义的字符属性
-   re.X(全拼：VERBOSE): 详细模式。这个模式下正则表达式可以是多行，忽略空白字符，并可以加入注释。


```
# -*- coding: utf-8 -*-
#一个简单的re实例，匹配字符串中的hello字符串
 
#导入re模块
import re
 
# 将正则表达式编译成Pattern对象，注意hello前面的r的意思是“原生字符串”
pattern = re.compile(r'hello')
 
# 使用Pattern匹配文本，获得匹配结果，无法匹配时将返回None
match1 = pattern.match('hello world!')
match2 = pattern.match('helloo world!')
match3 = pattern.match('helllo world!')
 
#如果match1匹配成功
if match1:
    # 使用Match获得分组信息
    print(match1.group())
else:
    print('match1匹配失败！')
 
 
#如果match2匹配成功
if match2:
    # 使用Match获得分组信息
    print(match2.group())
else:
    print('match2匹配失败！')
 
 
#如果match3匹配成功
if match3:
    # 使用Match获得分组信息
    print(match3.group())
else:
    print('match3匹配失败！')
```

## 2、Match 

 Match对象是一次匹配的结果，包含了很多关于此次匹配的信息，可以使用Match提供的可读属性或方法来获取这些信息。 

#### 属性：

1. ##### string: 匹配时使用的文本。

2. ##### re: 匹配时使用的Pattern对象。

3. ##### pos: 文本中正则表达式开始搜索的索引。值与Pattern.match()和Pattern.seach()方法的同名参数相同。

4. ##### endpos: 文本中正则表达式结束搜索的索引。值与Pattern.match()和Pattern.seach()方法的同名参数相同。

5. ##### lastindex: 最后一个被捕获的分组在文本中的索引。如果没有被捕获的分组，将为None。

6. ##### lastgroup: 最后一个被捕获的分组的别名。如果这个分组没有别名或者没有被捕获的分组，将为None。

#### 方法： 

1. ##### group([group1, …])：

   获得一个或多个分组截获的字符串；指定多个参数时将以元组形式返回。group1可以使用编号也可以使用别名；编号0代表整个匹配的子串；不填写参数时，返回group(0)；没有截获字符串的组返回None；截获了多次的组返回最后一次截获的子串。

2. ##### groups([default])：

   以元组形式返回全部分组截获的字符串。相当于调用group(1,2,…last)。default表示没有截获字符串的组以这个值替代，默认为None。

3. ##### groupdict([default])：

   返回以有别名的组的别名为键、以该组截获的子串为值的字典，没有别名的组不包含在内。default含义同上。

4. ##### start([group])：

   返回指定的组截获的子串在string中的起始索引（子串第一个字符的索引）。group默认值为0。

5. ##### end([group])：

   返回指定的组截获的子串在string中的结束索引（子串最后一个字符的索引+1）。group默认值为0。

6. ##### span([group])：

   返回(start(group), end(group))。

7. ##### expand(template)：

   将匹配到的分组代入template中然后返回。template中可以使用\id或\g<id>、\g<name>引用分组，但不能使用编号0。\id与\g<id>是等价的；但\10将被认为是第10个分组，如果你想表达\1之后是字符'0'，只能使用\g<1>0

```
# encoding: UTF-8

import re

# 将正则表达式编译成Pattern对象

pattern = re.compile(r'hello')

# 使用Pattern匹配文本，获得匹配结果，无法匹配时将返回None

match = pattern.match('hello world!')

if match:
    # 使用Match获得分组信息
    print match.group()

### 输出 ###

# hello
```


```
# -*- coding: utf-8 -*-
#一个简单的match实例
 
import re
# 匹配如下内容：单词+空格+单词+任意字符
m = re.match(r'(\w+) (\w+)(?P<sign>.*)', 'hello world!')
 
print "m.string:", m.string
print "m.re:", m.re
print "m.pos:", m.pos
print "m.endpos:", m.endpos
print "m.lastindex:", m.lastindex
print "m.lastgroup:", m.lastgroup
 
print "m.group():", m.group()
print "m.group(1,2):", m.group(1, 2)
print "m.groups():", m.groups()
print "m.groupdict():", m.groupdict()
print "m.start(2):", m.start(2)
print "m.end(2):", m.end(2)
print "m.span(2):", m.span(2)
print r"m.expand(r'\g<2> \g<1>\g<3>'):", m.expand(r'\2 \1\3')
 
### output ###
# m.string: hello world!
# m.re: <_sre.SRE_Pattern object at 0x016E1A38>
# m.pos: 0
# m.endpos: 12
# m.lastindex: 3
# m.lastgroup: sign
# m.group(1,2): ('hello', 'world')
# m.groups(): ('hello', 'world', '!')
# m.groupdict(): {'sign': '!'}
# m.start(2): 6
# m.end(2): 11
# m.span(2): (6, 11)
# m.expand(r'\2 \1\3'): world hello!2
```

```
# -*- coding: utf-8 -*-

#两个等价的re匹配,匹配一个小数
import re

a = re.compile(r"""\d +  # the integral part
                   \.    # the decimal point
                   \d *  # some fractional digits""", re.X)

b = re.compile(r"\d+\.\d*")

match11 = a.match('3.1415')
match12 = a.match('33')
match21 = b.match('3.1415')
match22 = b.match('33') 

if match11:
    # 使用Match获得分组信息
    print match11.group()
else:
    print u'match11不是小数'
    
if match12:
    # 使用Match获得分组信息
    print match12.group()
else:
    print u'match12不是小数'
    
if match21:
    # 使用Match获得分组信息
    print match21.group()
else:
    print u'match21不是小数'

if match22:
    # 使用Match获得分组信息
    print match22.group()
else:
    print u'match22不是小数'
```

### 3、search

> search(string[, pos[, endpos]]) | re.search(pattern, string[, flags]):
> 这个方法用于查找字符串中可以匹配成功的子串。 

> match()函数只检测re是不是在string的开始位置匹配，
>
> search()会扫描整个string查找匹配，

> match（）只有在0位置匹配成功的话才有返回，如果不是开始位置匹配成功的话，match()就返回none
> 例如：
> print(re.match(‘super’, ‘superstition’).span())
>
> 会返回(0, 5)
>
> print(re.match(‘super’, ‘insuperable’))
>
> 则返回None
>
> search()会扫描整个字符串并返回第一个成功的匹配
> 例如：
>
> print(re.search(‘super’, ‘superstition’).span())
>
> 返回(0, 5)
> print(re.search(‘super’, ‘insuperable’).span())
>
> 返回(2, 7)

```
# -*- coding: utf-8 -*-
#一个简单的search实例
 
import re
 
# 将正则表达式编译成Pattern对象
pattern = re.compile(r'world')
 
# 使用search()查找匹配的子串，不存在能匹配的子串时将返回None
# 这个例子中使用match()无法成功匹配
match = pattern.search('hello world!')
 
if match:
    # 使用Match获得分组信息
    print match.group()
 
### 输出 ###
# world
```

### 4、split

> split(string[, maxsplit]) | re.split(pattern, string[, maxsplit]):
> 按照能够匹配的子串将string分割后返回列表。
>
> maxsplit用于指定最大分割次数，不指定将全部分割。

```
import re
 
p = re.compile(r'\d+')
print p.split('one1two2three3four4')
 
### output ###
# ['one', 'two', 'three', 'four', '']
```

### 5、findall

>  findall(string[, pos[, endpos]]) | re.findall(pattern, string[, flags]):
> 搜索string，以列表形式返回全部能匹配的子串。 

```
import re
 
p = re.compile(r'\d+')
print p.findall('one1two2three3four4')
 
### output ###
# ['1', '2', '3', '4']
```

### 6、finditer

>  sub(repl, string[, count]) | re.sub(pattern, repl, string[, count]):
> 使用repl替换string中每一个匹配的子串后返回替换后的字符串。
> 当repl是一个字符串时，可以使用\id或\g<id>、\g<name>引用分组，但不能使用编号0。
> 当repl是一个方法时，这个方法应当只接受一个参数（Match对象），并返回一个字符串用于替换（返回的字符串中不能再引用分组）。
> count用于指定最多替换次数，不指定时全部替换。 

```
import re
 
p = re.compile(r'(\w+) (\w+)')
s = 'i say, hello world!'
 
print p.sub(r'\2 \1', s)
 
def func(m):
    return m.group(1).title() + ' ' + m.group(2).title()
 
print p.sub(func, s)
 
### output ###
# say i, world hello!
# I Say, Hello World!
```

### 7、subn