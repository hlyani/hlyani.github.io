# stratos-ui 相关

##### 1、构建环境

```
cd / git clone https://github.com/SUSE/stratos-ui.git
```

##### 2、安装nodejs

```
yum -y install gcc gcc-c++

# nodejs v6.11.4

cd /usr/local/src/ wget http://nodejs.org/dist/v6.11.4/node-v6.11.4.tar.gz tar -zxvf node-v6.11.4.tar.gz ./configure –prefix=/usr/local/src/node-v6.11.4 make make install

# yum install epel-release #yum install nodejs #yum install npm

vim /etc/profile export NODE_HOME=/usr/local/src/node-v6.11.4/ export PATH=$NODE_HOME/bin:$PATH

source /etc/profile

node –version npm –version
```

##### 3、编译安装go

```
# glide v0.13.0 #go 1.8.4

wget https://storage.googleapis.com/golang/go1.8.4.linux-amd64.tar.gz tar -C /usr/local -zxvf go1.8.4.linux-amd64.tar.gz

go get github.com/Masterminds/glide #go install github.com/Masterminds/glide cp ./go/bin/glide /usr/local/go/bin/

vim /etc/profile export NODE_HOME=/usr/local/src/node-v6.11.4/ export GOPATH=/usr/local/go export PATH=$NODE_HOME/bin:$GOPATH/bin:$PATH

source /etc/profile
```

##### 4、编译stratos-ui(npm run build-backend需要翻墙)

```
cd /stratos-ui

npm install -g gulp bower bower install –allow-root npm install –only=prod npm run build npm run build-backend npm run build-cf

npm run build-backend && npm run build-cf

CERTS_PATH=/stratos-ui/dev-certs ./generate_cert.sh

echo 'DATABASE_PROVIDER=sqlite HTTP_CONNECTION_TIMEOUT_IN_SECS=10 HTTP_CLIENT_TIMEOUT_IN_SECS=20 CONSOLE_PROXY_TLS_ADDRESS=:443 CF_ADMIN_ROLE=cloud_controller.admin CF_CLIENT=cf ALLOWED_ORIGINS=http://nginx SESSION_STORE_SECRET=wheeee! ENCRYPTION_KEY=B374A26A71490437AA024E4FADD5B497FDFF1A8EA6FF12F6FB65AF2720B59CCF CONSOLE_ADMIN_SCOPE=openid UAA_ENDPOINT=https://uaa.cf.tmp.com CONSOLE_CLIENT=cf SKIP_SSL_VALIDATION=true SQLITE_KEEP_DB=true AUTO_REG_CF_URL=https://api.cf.tmp.com'| tee > /stratos-ui/config.properties

chmod +x portal-proxy ./portal-proxy
```
