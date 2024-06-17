# 测试工具

# 一、ab（Apache Benchmark）

```
apt -y install apache2-utils
```

```
ab -n 1000 -c 100 http://你的网站地址/

    -n 1000：表示总共要发送1000个请求。
    -c 100：表示模拟100个并发用户，也就是同时有100个用户在访问网站。
命令执行后，ab会输出一些测试结果，包括每秒处理的请求数（RPS）、平均响应时间等。
```

```
Server Software:        nginx/1.10.2 (服务器软件名称及版本信息)
Server Hostname:        192.168.1.106(服务器主机名)
Server Port:            80 (服务器端口)
Document Path:          /index1.html. (供测试的URL路径)
Document Length:        3721 bytes (供测试的URL返回的文档大小)
Concurrency Level:      1000 (并发数)
Time taken for tests:   2.327 seconds (压力测试消耗的总时间)
Complete requests:      5000 (的总次数)
Failed requests:        688 (失败的请求数)
Write errors:           0 (网络连接写入错误数)
Total transferred:      17402975 bytes (传输的总数据量)
HTML transferred:       16275725 bytes (HTML文档的总数据量)
Requests per second:    2148.98 [#/sec] (mean) (平均每秒的请求数) 这个是非常重要的参数数值，服务器的吞吐量 
Time per request:       465.338 [ms] (mean) (所有并发用户(这里是1000)都请求一次的平均时间)
Time  request:       0.247 [ms] (mean, across all concurrent requests) (单个用户请求一次的平均时间)
Transfer rate:          7304.41 [Kbytes/sec] received 每秒获取的数据长度 (传输速率，单位：KB/s)
...
Percentage of the requests served within a certain time (ms)
  50%    347  ## 50%的请求在347ms内返回 
  66%    401  ## 60%的请求在401ms内返回 
  75%    431
  80%    516
  90%    600
  95%    846
  98%   1571
  99%   1593
 100%   1619 (longest request)

其他Linux系统的性能指标，如CPU负载、内存使用、网络带宽等。
```

