# Minio

[https://github.com/minio/minio.git](https://github.com/minio/minio.git)

# 一、安装

[https://github.com/minio/minio](https://github.com/minio/minio)



```
docker run -p 9090:9000 --name minio \
  -v /etc/localtime:/etc/localtime \
  -v /data/minio/data:/data \
  -v /data/minio/config:/root/.minio \
  -d minio/minio server /data
```

```
http://127.0.0.1:9090/
AccessKey: minioadmin SecretKey: minioadmin
```

# 二、使用

```
mc alias set local https://test.com:59000 username password
mc admin info local
mc ls local/aaa
mc cp local/aaa/file1 .
mc put ./test local/aaa
mc get local/aaa/file1 ./
```

```
curl https://test.com:59000/aaa/file1 -o file1
curl -k -C- -O --retry 3 https://test.com:59000/aaa/file1
```

```
curl -X PUT -u "<username>:<password>" -T "example.txt" \
     http://<minio-host>:<port>/mybucket/example.txt
curl -X GET -u "<username>:<password>" \
     http://<minio-host>:<port>/mybucket/example.txt -o "example.txt"
curl -X DELETE -u "<username>:<password>" \
     http://<minio-host>:<port>/mybucket/example.txt
curl -X GET -u "<username>:<password>" \
     http://<minio-host>:<port>/mybucket?list-type=2
```

mc 使用

| 命令    | 作用                                          |
| ------- | --------------------------------------------- |
| ls      | 列出文件和文件夹                              |
| mb      | 创建一个存储桶或一个文件夹                    |
| cat     | 显示文件和对象内容                            |
| pipe    | 将一个STDIN重定向到一个对象或者文件或者STDOUT |
| share   | 生成用于共享的URL                             |
| cp      | 拷贝文件和对象                                |
| mirror  | 给存储桶和文件夹做镜像                        |
| find    | 基于参数查找文件                              |
| diff    | 对两个文件夹或者存储桶比较差异                |
| rm      | 删除文件和对象                                |
| events  | 管理对象通知                                  |
| watch   | 监听文件和对象的事件                          |
| policy  | 管理访问策略                                  |
| session | 为cp命令管理保存的会话                        |
| config  | 管理mc配置文件                                |
| update  | 检查软件更新                                  |
| version | 输出版本信息                                  |

运行 MinIO Client

```
docker run -it --entrypoint=/bin/sh minio/mc
```



```
mc config host add minio http://172.20.32.232:9090 minioadmin minioadmin --api s3v4
mc ls minio               //查看存储桶
mc ls minio/test       //查看存储桶test中存在的文件
mc mb minio/dnps                                       //创建一个名为dnps的存储桶

mc share download minio/test/small.jpg       //共享test桶下small.jpg文件的下载路径

mc find minio/test --name "*.jpg"                    //查找test存储桶中的png文件

mc policy set download minio/dnps/             //设置权限：none, download, upload, public

mc policy list minio/dnps/                              //查看存储桶当前权限

mc cp minio/test/small.jpg   minio/dnps/        //拷贝文件和对象
```

