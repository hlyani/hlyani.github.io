# 常用软件安装

##1、rocket
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


##2、samba
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


##3、gitlab
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

##4、wiki
[https://www.dokuwiki.org/dokuwiki](https://www.dokuwiki.org/dokuwiki)
[https://github.com/bitnami/bitnami-docker-dokuwiki](https://github.com/bitnami/bitnami-docker-dokuwiki)

```
docker run -d -p 80:80 -p 443:443 --name dokuwiki \
 -e DOKUWIKI_PASSWORD=qwe \
 --volume /srv/dokuwiki:/bitnami/dokuwiki \
 bitnami/dokuwiki:latest
```
> 可用参数

```
DOKUWIKI_USERNAME: Dokuwiki application SuperUser name. Default: superuser
DOKUWIKI_FULL_NAME: Dokuwiki SuperUser Full Name. Default: Full Name
DOKUWIKI_PASSWORD: Dokuwiki application password. Default: bitnami1
DOKUWIKI_EMAIL: Dokuwiki application email. Default: user@example.com
DOKUWIKI_WIKI_NAME: Dokuwiki wiki name. Default: Bitnami DokuWiki
```

## 5、redmine

[https://hub.docker.com/_/redmine](https://hub.docker.com/_/redmine)

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

