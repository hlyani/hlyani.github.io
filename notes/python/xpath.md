# xpath 相关

 		XPath，全称 XML Path Language，既 XML  路径语言，它是一门在 XML 文档中查找信息的语言。同样适用于 HTML 文档内容的搜索。

[官方文档](https://www.w3.org/TR/xpath/)

### 一、xpath 语法

##### 1、常用规则

| 路径表达式   | 描述                                                     |
| -------- | -------------------------------------------------------- |
| nodename | 选取此节点的所有子节点                                   |
| /        | 从根节点选取                                             |
| //       | 从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置 |
| .        | 选取当前节点                                             |
| ..       | 选取当前节点的父节点                                     |
| @        | 选取属性                                                 |

##### 2、xpath 运算符

| 运算符 | 描述           | 实例              | 返回值                               |
| :----- | :------------- | :---------------- | :----------------------------------- |
| or     | 或             | age=18 or age=20  | age=18：True；age=21：False          |
| and    | 与             | age>18 and age<21 | age=20：True；age=21：False          |
| mod    | 计算除法的余数 | 5 mod 2           | 1                                    |
| \|     | 计算两个节点集 | //book \| //cd    | 返回所有拥有 book 和 cd 元素的节点集 |
| +      | 加法           | 5 + 3             | 8                                    |
| -      | 减法           | 5 - 3             | 2                                    |
| *      | 乘法           | 5 * 3             | 15                                   |
| div    | 除法           | 8 div 4           | 2                                    |
| =      | 等于           | age=19            | 判断简单，不再赘述                   |
| !=     | 不等于         | age!=19           | 判断简单，不再赘述                   |
| <      | 小于           | age<19            | 判断简单，不再赘述                   |
| <=     | 小于等于       | age<=19           | 判断简单，不再赘述                   |
| >      | 大于           | age>19            | 判断简单，不再赘述                   |
| >=     | 大于等于       | age>=19           | 判断简单，不再赘述                   |

##### 3、实例

| 路径表达式      | 结果                                                         |
| :-------------- | :----------------------------------------------------------- |
| bookstore       | 选取 bookstore 元素的所有子节点。                            |
| /bookstore      | 选取根元素 bookstore。注释：假如路径起始于正斜杠( / )，则此路径始终代表到某元素的绝对路径！ |
| bookstore/book  | 选取属于 bookstore 的子元素的所有 book 元素。                |
| //book          | 选取所有 book 子元素，而不管它们在文档中的位置。             |
| bookstore//book | 选择属于 bookstore 元素的后代的所有 book 元素，而不管它们位于 bookstore 之下的什么位置。 |
| //@lang         | 选取名为 lang 的所有属性。                                   |
| /bookstore/book[1]                 | 选取属于 bookstore 子元素的第一个 book 元素。                |
| /bookstore/book[last()]            | 选取属于 bookstore 子元素的最后一个 book 元素。              |
| /bookstore/book[last()-1]          | 选取属于 bookstore 子元素的倒数第二个 book 元素。            |
| /bookstore/book[position()<3]      | 选取最前面的两个属于 bookstore 元素的子元素的 book 元素。    |
| //title[@lang]                     | 选取所有拥有名为 lang 的属性的 title 元素。                  |
| //title[@lang='eng']               | 选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。   |
| /bookstore/book[price>35.00]       | 选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。 |
| /bookstore/book[price>35.00]/title | 选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。 |
| *      | 匹配任何元素节点。   |
| @*     | 匹配任何属性节点。   |
| node() | 匹配任何类型的节点。 |
| /bookstore/* | 选取 bookstore 元素的所有子元素。 |
| //*          | 选取文档中的所有元素。            |
| //title[@*]  | 选取所有带有属性的 title 元素。   |
| //book/title \| //book/price     | 选取 book 元素的所有 title 和 price 元素。                   |
| //title \| //price               | 选取文档中的所有 title 和 price 元素。                       |
| /bookstore/book/title \| //price | 选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。 |

### 二、实例

##### 1、安装

```
pip install lxml
```

##### 2、编写代码实例

```xml
from lxml import etree

text = '''
<div>
<ul>
<li class="item-0"><a href="link1.html"><span>first item</span></a></li>
<li class="item-1"><a href="link2.html">second item</a></li>
<li class="item-inactive"><a href="link3.html">third item</a></li>
<li class="item-1"><a href="link4.html">fourth item</a></li>
    <li class="item-0"><a href="link5.html">fifth item</a></li>
</ul>
</div>
'''

html = etree.HTML(text)
# 获取所有祖先节点
result = html.xpath('//li[1]/ancestor::*')
print(result)
# 获取 div 祖先节点
result = html.xpath('//li[1]/ancestor::div')
print(result)
# 获取当前节点所有属性值
result = html.xpath('//li[1]/attribute::*')
print(result)
# 获取 href 属性值为 link1.html 的直接子节点
result = html.xpath('//li[1]/child::a[@href="link1.html"]')
print(result)
# 获取所有的的子孙节点中包含 span 节点但不包含 a 节点
result = html.xpath('//li[1]/descendant::span')
print(result)
# 获取当前所有节点之后的第二个节点
result = html.xpath('//li[1]/following::*[2]')
print(result)
# 获取当前节点之后的所有同级节点
result = html.xpath('//li[1]/following-sibling::*')
print(result)

"""
[<Element html at 0x231a8965388>, <Element body at 0x231a8965308>, <Element div at 0x231a89652c8>, <Element ul at 0x231a89653c8>]
[<Element div at 0x231a89652c8>]
['item-0']
[<Element a at 0x231a89653c8>]
[<Element span at 0x231a89652c8>]
[<Element a at 0x231a89653c8>]
[<Element li at 0x231a8965308>, <Element li at 0x231a8965408>, <Element li at 0x231a8965448>, <Element li at 0x231a8965488>]
"""
```