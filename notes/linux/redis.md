# redis

# 一、配置

在 `redis.conf` 启用 LRU 过期策略：

```
maxmemory-policy allkeys-lru
```

# 二、基础使用

```
redis-cli -h 127.0.0.1 -p 6379 --user redis -a infini_rag_flow_helm ping
```

```
redis-cli
127.0.0.1:6379> auth infini_rag_flow_helm
OK
```

键值操作

```
SET key "hello"   # 设置 key
GET key           # 获取 key
DEL key           # 删除 key
EXPIRE key 10     # 设置 key 10 秒后过期
```

列表

```
LPUSH list val1   # 从左侧插入
RPUSH list val2   # 从右侧插入
LPOP list         # 从左侧弹出
RPOP list         # 从右侧弹出
LRANGE list 0 -1  # 获取所有列表元素
```

哈希

```
HSET user:1 name "Alice" age 25
HGET user:1 name
HGETALL user:1
```

集合

```
SADD myset "a" "b" "c"
SMEMBERS myset
SISMEMBER myset "a"
```

有序集合

```
ZADD scores 100 "Alice" 200 "Bob"
ZRANGE scores 0 -1 WITHSCORES
```

查看所有key

```
select 1

keys *

scan 0
SCAN 0 MATCH he*

redis-cli --scan --pattern "*"
```

```
redis-cli -n 1
dbsize    # 查看一共多少key
INFO memory
FLUSHDB   # 只清空当前数据库
FLUSHALL  # 清空 Redis 所有数据库
```

```
TYPE task_consumer_0
```

> `string`  → 代表是普通字符串（可用 `GET` 获取）。
>
> `list`    → 代表是列表（可用 `LRANGE` 获取）。
>
> `set`     → 代表是集合（可用 `SMEMBERS` 获取）。
>
> `hash`    → 代表是哈希表（可用 `HGETALL` 获取）。
>
> `zset`    → 代表是有序集合（可用 `ZRANGE` 获取）。

```
ZRANGE task_consumer_0 0 -1  # zset
LRANGE task_consumer_0 0 -1  # list
SMEMBERS task_consumer_0     # set
HGETALL task_consumer_0      # hash
XREVRANGE rag_flow_svr_queue + - COUNT 5 # stream  最新 5 条消息
XREAD COUNT 10 STREAMS rag_flow_svr_queue 0
XREAD BLOCK 0 STREAMS rag_flow_svr_queue $ # 持续监听新消息
```

| 操作                | 命令                                                         |
| ------------------- | ------------------------------------------------------------ |
| 查看 Stream 长度    | `XLEN rag_flow_svr_queue`                                    |
| 查看所有消息        | `XRANGE rag_flow_svr_queue - +`                              |
| 查看最新的 N 条消息 | `XREVRANGE rag_flow_svr_queue + - COUNT N`                   |
| 监听 Stream 消息    | `XREAD BLOCK 0 STREAMS rag_flow_svr_queue $`                 |
| 读取并标记已消费    | `XREADGROUP GROUP mygroup myconsumer COUNT 1 STREAMS rag_flow_svr_queue >` |
| 删除特定消息        | `XDEL rag_flow_svr_queue <message_id>`                       |
| 清理旧消息          | `XTRIM rag_flow_svr_queue MAXLEN 1000`                       |

# 三、编码

```
import redis

r = redis.Redis(host='localhost', port=6379, decode_responses=True)

r.set("username", "Alice", ex=10)  # 10 秒后过期
print(r.get("username"))  # Alice
```

redis事务

```
MULTI
SET key "value"
INCR counter
EXEC
```

```
MULTI         # 开启事务
SET key1 "A"  # 设置 key1=A
SET key2 "B"  # 设置 key2=B
INCR counter  # 递增 counter
EXEC          # 执行事务
```

redis发布/订阅(Pub/Sub)

```
PUBLISH mychannel "Hello, World!"
SUBSCRIBE mychannel
```

redis分布式锁

```
SET mylock "locked" EX 10 NX  # 10 秒自动过期
```

```
lock = r.set("mylock", "1", ex=10, nx=True)
if lock:
    print("获取锁成功")
else:
    print("锁已被占用")
```

