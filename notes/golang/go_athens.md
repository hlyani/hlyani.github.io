# Go Authens

# 一、安装

```
docker run -d --restart=always \
    -p 3000:3000 \
    -v ${PWD}/config.toml:/config/config.toml \
    -v ${PWD}/goproxy:/root/go \
    -e all_proxy=http://192.168.0.1:1080  \
    -e https_proxy=http://192.168.0.1:1080  \
    -e http_proxy=http://192.168.0.1:1080  \
    --name athens \
    gomods/athens:latest
```

## 1、config.toml

```
GoBinaryEnvVars = ["GOPROXY=https://goproxy.cn", "GOSUMDB=off"]
GoGetWorkers = 100
StorageType = "disk"
#GlobalEndpoint = "https://goproxy.cn,https://proxy.golang.org,https://goproxy.io,direct"
#GlobalEndpoint = "https://proxy.golang.org,direct"
#GlobalEndpoint = "https://proxy.golang.org,direct"
#GlobalEndpoint = "https://goproxy.io,direct"
#GlobalEndpoint = "https://proxy.golang.org"
GlobalEndpoint = "https://mirrors.aliyun.com/goproxy/"
#SumDBs = ["https://sum.golang.org"]
SumDBs = ["https://sum.golang.google.cn/"]
#DownloadMode = "sync"
DownloadMode = "file:/root/go/download.toml"
[Storage]
    [Storage.Disk]
        RootPath = "/root/go"
```

## 2、/root/go/download.toml

```
downloadURL = "https://mirrors.aliyun.com/goproxy/"

mode = "sync"

download "golang.zx2c4.com/go118/*" {
    mode = "redirect"
    downloadURL = "https://proxy.golang.org"
}
```

# 二、使用

## 1、浏览器查看已同步包

```
http://192.168.0.10:3000/catalog
```

## 2、使用

```
# export GOPROXY=http://192.168.0.10:3000
go env -w GOPROXY=http://192.168.0.10:3000
go env -w GOSUMDB=off
```

