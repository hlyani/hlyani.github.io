# Vim 常用命令

##### 1、set
```
忽略大小写查找
:set ignorecase
:set noignorecase

高亮搜索结果
:set hlsearch
:set nohlsearch

:e ++enc=utf8 filename, 让vim用utf-8的编码打开这个文件。
:w ++enc=gbk，不管当前文件什么编码，把它转存成gbk编码。
:set fenc或:set fileencoding，查看当前文件的编码。
```
##### 2、快捷键
```
w 光标移动向后一个单词，单词首，2w向后两个
e 光标移动向后一个单词，单词尾,2e
ge 和e相反，向前
b 光标移动向前一个单词，单词首，2b
^ 移动到本行第一个非空白字符上
0、HOME 移动到本行第一个字符上
$ 移动到行尾， 3$ 移动到下面3行的行尾
gg 文件头
G 文件尾
f fx找到下一个为x的字符
:20 相当于 20G ，跳到指定行
Ctrl + e 向下滚动一行
Ctrl + y 向上滚动一行
Ctrl + d 向下滚动半屏
Ctrl + u 向上滚动半屏
Ctrl + f 向下滚动一屏
Ctrl + b 向上滚动一屏
u 撤销（undo）
U 撤销对整行的操作
Ctrl + r 重做（redo)
3x 删除当前光标开始向后的三个字符
dd 删除当前行
dj 删除上一行
dk 删除下一行
```
##### 3、执行shell
```
:!command
:!ls 列出当前目录下文件
:!perl -c script.pl 检查perl脚本语法，可以不用退出vim，非常方便。
:!perl script.pl 执行perl脚本，可以不用退出vim，非常方便。
:suspend或Ctrl - Z 挂起vim，回到shell，按fg可以返回vim。
```
##### 4、注释命令
```
3,5 s/^/#/g 注释第3-5行
3,5 s/^#//g 解除3-5行的注释
1,$ s/^/#/g 注释整个文档。
:%s/^/#/g 注释整个文档，此法更快。
```
##### 5、宏
```
. 重复上一个编辑动作
qa 开始录制宏a（键盘操作记录）
q 停止录制
@a 播放宏a
```
##### 6、文件加解密
```
vim -x file 开始编辑一个加密的文件。
:X 为当前文件设置密码。
:set key= 去除文件的密码。
```

##### 7、常用配置

```
vim ~/.vimrc

set nobackup
set noswapfile
set nowritebackup
```

