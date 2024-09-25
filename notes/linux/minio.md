# Minio

[https://github.com/minio/minio.git](https://github.com/minio/minio.git)

# 一、安装

[https://github.com/minio/minio](https://github.com/minio/minio)

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

