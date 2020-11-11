# chartmuseum 相关

[chartmuseum github](https://github.com/helm/chartmuseum)

# 一、安装 chartmuseum

## 1、添加 helm 源

```
helm repo add stable https://charts.helm.sh/stable
```

## 2、拉取 chart 包

```
helm pull stable/chartmuseum --untar
```

## 3、修改配置

```
vim chartmuseum/value.yaml

DISABLE_API: false
type: NodePort
```

## 4、部署

```
helm install chartmuseum/
```

# 二、通过 helm 添加使用

```
helm repo add chartmuseum http://localhost:8080
```

```
helm search repo chartmuseum
```

```
helm install chartmuseum/mychart
```

```
helm push mychart/ chartmuseum
```

# 三、通过 curl 命令，进行 CRUD 操作

## 1、查看仓库信息

```
curl http://localhost:8080/index.yaml
```

## 2、查看所有软件

```
curl http://localhost:8080/api/charts
```

## 3、查看某个软件的所有版本信息

```
curl http://localhost:8080/api/charts/nginx
```

## 4、查看某个软件的具体版本的信息

```
curl http://localhost:8080/api/charts/nginx/5.1.5
```

## 5、下载软件包

```
curl -O http://localhost:8080/charts/nginx-5.1.5.tgz
```

## 6、上传软件包到仓库

```
curl --data-binary "@rancher-2.5.1.tgz" http://localhost:8080/api/charts

curl -X POST -k --data-binary "@mychart-1.0-0.1.1.tgz" localhost:8080/api/charts
```

## 7、删除仓库中的软件

```
curl -X DELETE http://localhost:8080/api/charts/nginx/5.1.5
```

