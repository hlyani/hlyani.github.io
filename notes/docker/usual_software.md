# 常用软件安装

# 1、rocket

[https://hub.docker.com/_/rocket-chat](https://hub.docker.com/_/rocket-chat)
[https://rocket.chat/install](https://rocket.chat/install)

```
docker run -d --name rocketchat-mongo mongo:4.0.10 --smallfiles --oplogSize 128 --replSet rs1 --storageEngine=mmapv1
```

```
docker exec -d rocketchat-mongo bash -c 'echo -e "replication:\n replSetName: \"rs01\"" | tee -a /etc/mongod.conf && mongo --eval "printjson(rs.initiate())"'
```

```
docker run -d --name rocketchat --link rocketchat-mongo -e "MONGO_URL=mongodb://rocketchat-mongo:27017/rocketchat" -e MONGO_OPLOG_URL=mongodb://rocketchat-mongo:27017/local?replSet=rs01 -e ROOT_URL=http://192.168.21.87:3001 -p 3001:3000 rocketchat/rocket.chat:1.2.1
```

# 2、samba

[https://github.com/dperson/samba](https://github.com/dperson/samba)

> ```
> -s "<name;/path>[;browse;readonly;guest;users;admins;writelist;comment]"
>     Configure a share
>     required arg: "<name>;</path>"
>     <name> is how it's called for clients
>     <path> path to share
>     NOTE: for the default values, just leave blank
>     [browsable] default:'yes' or 'no'
>     [readonly] default:'yes' or 'no'
>     [guest] allowed default:'yes' or 'no'
>     NOTE: for user lists below, usernames are separated by ','
>     [users] allowed default:'all' or list of allowed users
>     [admins] allowed default:'none' or list of admin users
>     [writelist] list of users that can write to a RO share
>     [comment] description of share
> ```

```
docker run -it --name samba -p 139:139 -p 445:445 \
  --restart always \
  -e TZ=EST5EDT \
  -v /fs/samba:/mount \
  -d dperson/samba -p \
  -s "samba;/mount/;yes;no;yes;all;all;all;all"
```

```
mkdir /opt/test
chmod 777 -R /opt/test
```

```
docker run -it -p 139:139 -p 445:445 --name samba -v /opt/test:/mount -d dperson/samba \
            -u "test;qwe" \
            -s "test;/mount/;yes;no;yes;all;all;all" \
            -w "WORKGROUP" \
            -g "force user= test" \
            -g "guest account= test"
```

# 3、gitlab
[https://docs.gitlab.com/omnibus/docker/](https://docs.gitlab.com/omnibus/docker/)

```
docker run --detach \
  --hostname gitlab.example.com \
  --publish 8443:443 --publish 8080:80 --publish 8022:22 \
  --name gitlab \
  --restart always \
  --volume /srv/gitlab/config:/etc/gitlab \
  --volume /srv/gitlab/logs:/var/log/gitlab \
  --volume /srv/gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```

> 默认帐户的用户名是root，第一次访问时，将被重定向到密码重置屏幕,登录后，您可以更改用户名。

```
gitlab-ctl reconfigure
gitlab-ctl restart
gitlab-ctl status
gitlab-ctl stop
gitlab-ctl tail                                  
gitlab-ctl stop unicorn
gitlab-ctl stop sideki
```

##### FAQ

> fail to initialize orm engine: Sqlstore::Migration failed err: unable to open database file

```
chmod 777 -R ./grafana/data
```

> 2020-10-15_07:07:14.07056 time="2020-10-15T07:07:14Z" level=fatal msg="find gitaly" error="open /var/opt/gitlab/gitaly/gitaly.pid: permission denied" wrapper=3997

```
chown 998 gitaly.pid
chgrp 988 gitaly.pid
```

# 4、wiki

[https://www.dokuwiki.org/dokuwiki](https://www.dokuwiki.org/dokuwiki)
[https://github.com/bitnami/bitnami-docker-dokuwiki](https://github.com/bitnami/bitnami-docker-dokuwiki)

```
docker run -d -p 9080:8080 -p 9443:8443 --restart=always --name dokuwiki \
 -e DOKUWIKI_USERNAME=admin \
 -e DOKUWIKI_PASSWORD=qwe \
 -e ALLOW_EMPTY_PASSWORD=yes \
 -v /fs/wiki/data:/bitnami/dokuwiki \
 bitnami/dokuwiki:latest
```
> 可用参数

```
DOKUWIKI_USERNAME: Dokuwiki application username. Default: user
DOKUWIKI_FULL_NAME: Dokuwiki application user full name. Default: Full Name
DOKUWIKI_PASSWORD: Dokuwiki application password. Default: bitnami1
DOKUWIKI_EMAIL: Dokuwiki application email. Default: user@example.com
DOKUWIKI_WIKI_NAME: Dokuwiki wiki name. Default: Bitnami DokuWiki
```

# 5、redmine

[https://hub.docker.com/redmine](https://hub.docker.com/redmine)

> use SQLite3

```
docker run -d --name some-redmine redmine
```

> use PostgreSQL

```
docker run -d --name some-postgres --network some-network -e POSTGRES_PASSWORD=secret -e POSTGRES_USER=redmine postgres

docker run -d --name some-redmine --network some-network -e REDMINE_DB_POSTGRES=some-postgres -e REDMINE_DB_USERNAME=redmine -e REDMINE_DB_PASSWORD=secret redmine
```

> use MySQL

```
docker run -d --name some-mysql --network some-network -e MYSQL_USER=redmine -e MYSQL_PASSWORD=secret -e MYSQL_DATABASE=redmine -e MYSQL_RANDOM_ROOT_PASSWORD=1 mysql:5.7

docker run -d --name some-redmine --network some-network -e REDMINE_DB_POSTGRES=some-postgres -e REDMINE_DB_USERNAME=redmine -e REDMINE_DB_PASSWORD=secret redmine
```

# 6、vsftpd

[https://github.com/panubo/docker-vsftpd](https://github.com/panubo/docker-vsftpd)

> 生成秘钥

```
openssl req -x509 -nodes -days 3650 -newkey rsa:1024 -keyout /opt/pub/vsftpd.pem -out /opt/pub/vsftpd.pem
```

> 运行容器

```
docker run -d \
    -p 21:21 -p 4559-4564:4559-4564 \
    -e FTP_USER=root -e FTP_PASSWORD=qwe \
    -v /home/vsftpd:/home/vsftpd \
    -v /var/log/ftp:/var/log \
    -v /opt/pub/vsftpd.pem:/etc/ssl/certs/vsftpd.crt:ro \
    -v /opt/pub/vsftpd.pem:/etc/ssl/private/vsftpd.key:ro \
    -v /home/vsftpd:/srv \
    --restart=always \
    docker.io/panubo/vsftpd vsftpd /etc/vsftpd_ssl.conf
```

# 7、jenkins

[https://www.jenkins.io/doc/book/installing/docker/](https://www.jenkins.io/doc/book/installing/docker/)

```
docker network create jenkins

docker run -d  \
  --restart always \
  --network jenkins \
  --network-alias docker \
  --name jenkins -u root \
  -p 8080:8080  \
  -v /opt/jenkins:/var/jenkins_home  \
  jenkinsci/blueocean
```

# 8、harbor

[https://github.com/goharbor/harbor](https://github.com/goharbor/harbor)

[http://hlyani.gitee.io/hlyani.github.io/notes/docker/harbor.html](http://hlyani.gitee.io/hlyani.github.io/notes/docker/harbor.html)

```
./prepare
./install.sh --with-chartmuseum

docker-compose down -v
docker-compose up -d
docker-compose stop -v
```

# 9、nextcloud

```
docker run -d \
    --restart always \
    --name nextcloud \
    -p 8000:80 \
    -v /data/nextcloud:/var/www/html \
    nextcloud
```

# 10、svn

```
docker run --restart always --name svn -d -v /root/dockers/svn:/var/opt/svn -p 3690:3690 garethflowers/svn-server
```

```
docker exec -it svn /bin/sh
```

```
svnadmin create svn
```

```
vim svnserve.conf
anon-access = none             # 匿名用户不可读写，也可设置为只读 read
auth-access = write            # 授权用户可写
password-db = passwd           # 密码文件路径，相对于当前目录
authz-db = authz               # 访问控制文件
realm = /var/opt/svn/svn       # 认证命名空间，会在认证提示界面显示，并作为凭证缓存的关键字，可以写仓库名称比如svn
```

```
vim passwd
[users]
# harry = harryssecret
# sally = sallyssecret
admin = 123456
```

```
vim authz
[groups]
owner = admin
[/]               # / 表示所有仓库
admin = rw        # 用户 admin 在所有仓库拥有读写权限
[svn:/]           # 表示以下用户在仓库 svn 的所有目录有相应权限
@owner = rw       # 表示 owner 组下的用户拥有读写权限
```

```
svn co svn://127.0.0.1:3690/svn
```

