# nodejs 相关

##### 1、安装nodejs
```
curl -sL https://rpm.nodesource.com/setup_10.x | sudo bash -
yum install -y nodejs
```
##### 2、查看版本
```
node --version
```
##### 3、使用淘宝npm源
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install SOMEPAKAGE
```
##### 4、默认安装目录
```
/usr/local/node/0.10.24/lib/node_modules/
```

##### 5、使用国内镜像加速npm

```
npm config set registry=https://registry.npm.taobao.org
```

##### 6、使用国内镜像加速yarn

```
yarn config set registry https://registry.npm.taobao.org
```

##### 7、使用国内源安装

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

##### 8、TODO

