# 一、gitbook 相关

> 集成了 GitBook、Git、Markdown 等功能，还支持将书籍同步到 gitbook.com 网站，使我们可以很方便地编辑和管理书籍。

### 1、安装nodejs

```
curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -
yum install -y nodejs
node --version
```

### 2、安装gitbook

```
npm install -g gitbook-cli
```

### 3、初始化

```
mkdir mybook
gitbook init
```

### 4、编辑SUMMARY.md

```
# 目录

* [前言](README.md)
* [第一章](Chapter1/README.md)
  * [第1节：衣](Chapter1/衣.md)
  * [第2节：食](Chapter1/食.md)
  * [第3节：住](Chapter1/住.md)
  * [第4节：行](Chapter1/行.md)
* [第二章](Chapter2/README.md)
* [第三章](Chapter3/README.md)
* [第四章](Chapter4/README.md)

```

##### 再次执行 gitbook init

### 5、运行服务浏览mybook

```
gitbook serve
or
gitbook serve --port 8888
```

### 6、构建html

```
gitbook build
```
##### 将构建书籍，默认生成html输出到_book目录中。

### 7、生成pdf、epub、mobi格式电子书
```
npm install -g ebook-convert

gitbook pdf ./ ./mybook.pdf

gitbook epub ./ ./mybook.epub

gitbook mobi ./ ./mybook.mobi
```
### 8、配置目录折叠
```
在主目录添加book.json
{
    "plugins":[
            "expandable-chapters"
    ]
}

gitbook install
or
直接使用命令安装
npm install -g gitbook-plugin-splitter
```

```
* [第一章](Chapter1/README.md)
  * [第1节：衣](Chapter1/衣.md)
  * [第2节：食](Chapter1/食.md)
  * [第3节：住](Chapter1/住.md)
  * [第4节：行](Chapter1/行.md)
```
### 9、常用插件

https://www.jianshu.com/p/427b8bb066e6
https://www.cnblogs.com/mingyue5826/p/10307051.html#autoid-2-3-0

### 10、gitbook serve 找不到 fontsettings.js

```
cd ~/.gitbook\versions\3.2.3\lib\output\website
vim copyPluginAssets.js
删除112行
```

