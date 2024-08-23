# yq

[https://github.com/mikefarah/yq/](https://github.com/mikefarah/yq/)

# 1、下载

```
wget https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 -O /usr/bin/yq
chmod +x /usr/bin/yq
```

# 2、测试

```
cat a.yaml 
hl:
  yani: bbb
```

```
yq .hl a.yaml
```

```
yq e -i '.hl.yani = "bbb"'  a.yaml
```

```
cat a.yaml | yq .hl
```

```
admin_key=$(yq '.deployment.admin.admin_key[0].key' conf/config.yaml | sed 's/"//g')
```

