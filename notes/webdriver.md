# webdriver 相关

## 一、准备
##### 1、环境要求
- Window 操作系统
- Python3.5.0-amd64(64位)
- Selenium 2.48.0
- Pycharm(2017.1.1)
##### 2、Selenium with Python中文文档
https://selenium-python-zh.readthedocs.io/en/latest/index.html
##### 3、谷歌驱动
https://sites.google.com/a/chromium.org/chromedriver/

##### 4、将驱动下载，解压，配置解压后的驱动位置
```
driver = webdriver.Chrome(r"D:\chromedriver\chromedriver.exe")
```
##### 5、安装selenium（又名webdriver）
```
pip install selenium
```
## 二、快速入门
##### 1、创建文件webdriver.py,输入以下内容
```
#导包
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

#创建一个webdriver实例
driver = webdriver.Chrome(r"D:\chromedriver\chromedriver.exe")

#打开填写的URL地址，直到页面加载完毕
driver.get("http://www.baidu.com")

#确认标题是否包含"百度一下"
#print(driver.title)
assert "百度一下" in driver.title

#通过name获取前端对象
#<input type="text" class="s_ipt" name="wd" id="kw" maxlength="100" autocomplete="off">
elem = driver.find_element_by_name("wd")
#print(elem)

#清除input输入框
elem.clear()

#发送按键 "hello"
elem.send_keys("hello")

#发送回车按键
elem.send_keys(Keys.RETURN)

#判断返回内容中是否包含 "百度百科"
assert "百度百科" not in driver.page_source

#关闭webdriver实例
driver.close()
```
##### 2、执行程序
```
python webdriver.py
```
## 三、其他相关
##### 1、获取html dom对象的常用方法
```
element = driver.find_element_by_id("passwd-id")
element = driver.find_element_by_name("passwd")
element = driver.find_element_by_xpath("//input[@id='passwd-id']")
```
##### 2、填写表格
```
element = driver.find_element_by_xpath("//select[@name='name']")
all_options = element.find_elements_by_tag_name("option")
for option in all_options:
    print("Value is: %s" % option.get_attribute("value"))
    option.click()
```
##### 3、选择
```
from selenium.webdriver.support.ui import Select
select = Select(driver.find_element_by_name('name'))
select.select_by_index(index)
select.select_by_visible_text("text")
select.select_by_value(value)
```
##### 4、取消OPTION选择
```
select = Select(driver.find_element_by_id('id'))
select.deselect_all()
```
##### 5、拖放
```
element = driver.find_element_by_name("source")
target = driver.find_element_by_name("target")

from selenium.webdriver import ActionChains
action_chains = ActionChains(driver)
action_chains.drag_and_drop(element, target).perform()
```
##### 6、查找元素
```
find_element_by_id
find_element_by_name
find_element_by_xpath
find_element_by_link_text
find_element_by_partial_link_text
find_element_by_tag_name
find_element_by_class_name
find_element_by_css_selector
```
##### 7、其余请参照官网文档
https://selenium-python-zh.readthedocs.io/en/latest/index.html


## 四、例子
``` python
#导包
from selenium import webdriver
from selenium.webdriver.common.keys import Keys

driver = webdriver.Chrome(r"D:\chromedriver\chromedriver.exe")
driver.get("http://www.baidu.com")
assert "百度一下" in driver.title
elem = driver.find_element_by_name("wd")
elem.clear()
elem.send_keys("hello")
elem.send_keys(Keys.RETURN)
assert "百度百科" not in driver.page_source
driver.close()
```