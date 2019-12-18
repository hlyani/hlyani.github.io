# apidoc 相关

[https://apidocjs.com/](https://apidocjs.com/)

##### 1、安装 nodejs 

略

##### 2、安装 apidoc

```
npm install apidoc -g
```

##### 3、将 npm bin 目录，写入环境变量

```
echo -e "export PATH=$(npm prefix -g)/bin:$PATH" >> ~/.bashrc && source ~/.bashrc
```

##### 4、在项目目录创建 apidoc.api

```
cat apidoc.json 
{
  "name": "libvirtapi",
  "version": "1.0.0",
  "description": "libvirtapi 接口文档",
  "title": "libvirtapi",
  "url" : "http://YOUR_IP:8778"
}
```

##### 5、在项目目录执行以下命令，生成文档

```
apidoc -i ./ -o apidoc
```

