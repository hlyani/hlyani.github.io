# linux 源相关

# 一、alpine

```
echo -e "http://192.168.0.90:81/alpine/alpine/v3.13/main/\nhttp://192.168.0.90:81/alpine/alpine/v3.13/community/" > /etc/apk/repositories
```

```
sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
```

```
sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories
```

```
sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories
```

# 二、ubuntu

```
sed -i "s@http://.*archive.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
sed -i "s@http://.*security.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g" /etc/apt/sources.list
```

## 1、x86

```
echo "deb DIB_RESOURCE DIB_RELEASE main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE main restricted universe multiverse

deb DIB_RESOURCE DIB_RELEASE-security main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE-security main restricted universe multiverse

deb DIB_RESOURCE DIB_RELEASE-updates main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE-updates main restricted universe multiverse

deb DIB_RESOURCE DIB_RELEASE-proposed main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE-proposed main restricted universe multiverse

deb DIB_RESOURCE DIB_RELEASE-backports main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE-backports main restricted universe multiverse" > /etc/apt/sources.list

DIB_RELEASE=focal
DIB_RESOURCE=http://mirrors.aliyun.com/ubuntu/
#DIB_RESOURCE=https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
#DIB_RESOURCE=http://mirrors.ustc.edu.cn/ubuntu/

sed -i "s#DIB_RESOURCE#${DIB_RESOURCE}#g" /etc/apt/sources.list
sed -i "s#DIB_RELEASE#${DIB_RELEASE}#g" /etc/apt/sources.list
```

```
echo "deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse" > /etc/apt/sources.list
```

## 2、arm64

```
echo "deb DIB_RESOURCE DIB_RELEASE main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE main restricted universe multiverse

deb DIB_RESOURCE DIB_RELEASE-security main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE-security main restricted universe multiverse

deb DIB_RESOURCE DIB_RELEASE-updates main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE-updates main restricted universe multiverse

deb DIB_RESOURCE DIB_RELEASE-proposed main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE-proposed main restricted universe multiverse

deb DIB_RESOURCE DIB_RELEASE-backports main restricted universe multiverse
deb-src DIB_RESOURCE DIB_RELEASE-backports main restricted universe multiverse" > /etc/apt/sources.list

DIB_RELEASE=focal
DIB_RESOURCE=https://mirrors.aliyun.com/ubuntu-ports/
#DIB_RESOURCE=https://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/
#DIB_RESOURCE=http://mirrors.ustc.edu.cn/ubuntu-ports/

sed -i "s#DIB_RESOURCE#${DIB_RESOURCE}#g" /etc/apt/sources.list
sed -i "s#DIB_RELEASE#${DIB_RELEASE}#g" /etc/apt/sources.list
```

