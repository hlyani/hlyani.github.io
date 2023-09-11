# nodejs 相关

[download](https://nodejs.org/zh-cn/download)

## 1、安装nodejs
```
curl -LO https://nodejs.org/dist/v18.17.1/node-v18.17.1-linux-x64.tar.xz
or
wget https://nodejs.org/dist/v18.17.1/node-v18.17.1-linux-x64.tar.xz

tar -xvf node-v18.17.1-linux-x64.tar.xz -C /usr/local/
ln -s /usr/local/node-v18.17.1-linux-x64/bin/npm   /usr/local/bin/ 
ln -s /usr/local/node-v18.17.1-linux-x64/bin/node   /usr/local/bin/
```
## 2、查看版本
```
node --version
```
## 3、使用淘宝npm源
```
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install SOMEPAKAGE
```
## 4、默认安装目录
```
/usr/local/node/0.10.24/lib/node_modules/
```

## 5、使用国内镜像加速npm

```
npm config set registry=https://registry.npm.taobao.org
```

## 6、使用国内镜像加速yarn

```
yarn config set registry https://registry.npm.taobao.org
```

## 7、使用国内源安装

```
npm install -g cnpm --registry=https://registry.npm.taobao.org
```

## 8、安装 apidoc 

```
npm install apidoc -g
echo -e "export PATH=$(npm prefix -g)/bin:$PATH" >> ~/.bashrc && source ~/.bashrc
```

## 9、TODO

```
。。。
```

