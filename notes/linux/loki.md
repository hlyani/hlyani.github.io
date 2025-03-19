# Loki



```
docker run -d --name=loki -p 3100:3100 grafana/loki:latest
```



健康检测

```
curl -s "http://localhost:3100/ready"
```



查询

```
curl -s "http://localhost:3100/loki/api/v1/query_range?query={job='varlogs'}&limit=10"
```

