# Redis

# 一、高可用部署











# 二、基础使用

连接

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

主要类型

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

# 三、常用命令

## 1.基础命令

| 命令                           | 说明                                                   |
| ------------------------------ | ------------------------------------------------------ |
| `ping`                         | 检查 Redis 是否在线，返回 `PONG`                       |
| `auth <password>`              | 登录认证（如果设置了密码）                             |
| `select <db>`                  | 选择数据库（默认是 0）                                 |
| `flushdb`                      | 清空当前数据库                                         |
| `flushall`                     | 清空所有数据库                                         |
| `keys <pattern>`               | 查找符合模式的 key（不建议用于生产环境）               |
| keys *                         |                                                        |
| `exists <key>`                 | 判断 key 是否存在                                      |
| `del <key>`                    | 删除指定 key                                           |
| `expire <key> <seconds>`       | 为 key 设置过期时间（单位：秒）                        |
| `ttl <key>`                    | 查看 key 剩余过期时间（-1: 永不过期；-2: 不存在）      |
| `type <key>`                   | 查看 key 的类型                                        |
| select 1                       | 切换到第 1 个数据库                                    |
| scan 0                         | 使用游标非阻塞遍历所有 key，推荐用于生产环境           |
| SCAN 0 MATCH he*               | 遍历匹配以 `he` 开头的 key                             |
| SCAN 0 MATCH user:* COUNT 100  | 每次返回最多 100 条匹配 `user:*` 的 key                |
| dbsize                         | 查看一共多少key                                        |
| info                           | 显示 Redis 的所有运行信息                              |
| Info memory                    | 显示 Redis 内存使用详情                                |
| info stats                     | 查看命中率、命令处理量等统计信息                       |
| info commandstats              | 查看各命令的调用次数和耗时                             |
| monitor                        | 实时打印所有操作命令（调试用，慎用）                   |
| slowlog get                    | 查看慢查询日志                                         |
| config get <param>             | 查看配置参数值，例如 `config get maxmemory`            |
| config set <param> <value>     | 动态修改 Redis 配置，例如 `config set maxmemory 512mb` |
| redis-cli --scan --pattern "*" | 简化的 scan 语法，便于在 shell 使用（默认使用游标）    |
| redis-cli -n 1                 | 使用 redis-cli 操作第 1 个数据库                       |

## 2.字符串(String)

| 命令                   | 说明             |
| ---------------------- | ---------------- |
| `set <key> <value>`    | 设置字符串       |
| `get <key>`            | 获取字符串       |
| `incr <key>`           | 自增（整数）     |
| `decr <key>`           | 自减（整数）     |
| `append <key> <value>` | 追加字符串       |
| `mset k1 v1 k2 v2`     | 批量设置多个键值 |
| `mget k1 k2`           | 批量获取多个值   |

## 3.哈希(Hash)

| 命令                         | 说明             | 示例                            |
| ---------------------------- | ---------------- | ------------------------------- |
| `hset <key> <field> <value>` | 设置 hash 字段   | HSET user:1 name "Alice" age 25 |
| `hget <key> <field>`         | 获取字段值       | HGET user:1 name                |
| `hgetall <key>`              | 获取所有字段和值 | HGETALL user:1                  |
| `hdel <key> <field>`         | 删除字段         |                                 |
| `hexists <key> <field>`      | 检查字段是否存在 |                                 |
| `hmget <key> f1 f2`          | 获取多个字段值   |                                 |
| `hmset <key> f1 v1 f2 v2`    | 批量设置多个字段 |                                 |

## 4.列表(List)

| 命令                  | 说明         |
| --------------------- | ------------ |
| `lpush <key> <value>` | 左侧插入     |
| `rpush <key> <value>` | 右侧插入     |
| `lpop <key>`          | 左侧弹出     |
| `rpop <key>`          | 右侧弹出     |
| `lrange <key> 0 -1`   | 获取所有元素 |
| `llen <key>`          | 获取列表长度 |

## 5.集合(Set)

| 命令                       | 说明             | 示例                   |
| -------------------------- | ---------------- | ---------------------- |
| `sadd <key> <member>`      | 添加元素         | SADD myset "a" "b" "c" |
| `srem <key> <member>`      | 移除元素         |                        |
| `smembers <key> `          | 获取所有成员     | SMEMBERS myset         |
| `sismember <key> <member>` | 判断成员是否存在 | SISMEMBER myset "a"    |
| `scard <key>`              | 获取集合元素数量 |                        |

## 6.有序集合(Sorted Set)

| 命令                          | 说明                       | 示例                              |
| ----------------------------- | -------------------------- | --------------------------------- |
| `zadd <key> <score> <member>` | 添加元素及分数             | ZADD scores 100 "Alice" 200 "Bob" |
| `zrem <key> <member>`         | 删除成员                   |                                   |
| `zrange <key> 0 -1`           | 获取所有成员（按分数升序） | ZRANGE scores 0 -1 WITHSCORES     |
| `zrevrange <key> 0 -1`        | 获取所有成员（按分数降序） |                                   |
| `zscore <key> <member>`       | 获取成员分数               |                                   |

## 7.发布订阅(Pub/Sub)

| 命令                          | 说明           |
| ----------------------------- | -------------- |
| `publish <channel> <message>` | 向频道发送消息 |
| `subscribe <channel>`         | 订阅频道       |
| `unsubscribe <channel>`       | 取消订阅       |

## 8.服务器相关

| 命令                         | 说明                       |
| ---------------------------- | -------------------------- |
| `info`                       | 查看服务器信息             |
| `monitor`                    | 实时查看所有请求（调试用） |
| `config get <param>`         | 获取配置项                 |
| `config set <param> <value>` | 动态修改配置项             |
| `save`                       | 同步保存快照到磁盘         |
| `bgsave`                     | 异步保存快照               |
| `lastsave`                   | 上一次成功保存时间戳       |
| `shutdown`                   | 停止 Redis 服务            |

## 9.RedisJSON

### 基础操作

| 命令                                 | 说明                                        |
| ------------------------------------ | ------------------------------------------- |
| `JSON.SET <key> <path> <json>`       | 设置 JSON 数据，`path` 通常是 `$`（表示根） |
| `JSON.GET <key> [path]`              | 获取 JSON 数据，默认获取全部                |
| `JSON.DEL <key> [path]`              | 删除 JSON 的某个路径                        |
| `JSON.TYPE <key> [path]`             | 获取指定路径的 JSON 类型                    |
| `JSON.MGET <key1> <key2> ... <path>` | 批量获取多个 key 的某个路径数据             |

------

### 修改操作

| 命令                                            | 说明                                 |
| ----------------------------------------------- | ------------------------------------ |
| `JSON.NUMINCRBY <key> <path> <num>`             | JSON 中数字值递增                    |
| `JSON.STRAPPEND <key> <path> <str>`             | 向 JSON 字符串追加内容               |
| `JSON.ARRAPPEND <key> <path> <element>`         | 向 JSON 数组追加元素                 |
| `JSON.ARRPOP <key> <path> [index]`              | 弹出 JSON 数组中的元素               |
| `JSON.ARRINSERT <key> <path> <index> <element>` | 在数组指定位置插入元素               |
| `JSON.ARRLEN <key> <path>`                      | 获取 JSON 数组长度                   |
| `JSON.ARRINDEX <key> <path> <element>`          | 获取元素在数组中的下标               |
| `JSON.CLEAR <key> [path]`                       | 清空某个 JSON 路径下的值（保留结构） |

> $ 表示根节点

```
# 设置一个 JSON 对象
JSON.SET user:1 $ '{"name": "Alice", "age": 30, "tags": ["dev", "redis"]}'

# 获取整个对象
JSON.GET user:1

# 获取某个字段
JSON.GET user:1 $.name

# 数组追加
JSON.ARRAPPEND user:1 $.tags '"admin"'

# 删除某字段
JSON.DEL user:1 $.age
```

# 四、优化

[sentinel配置](https://github.com/redis/redis/blob/unstable/sentinel.conf)

[redis配置](https://github.com/redis/redis/blob/unstable/redis.conf)

```
port 6379
bind 0.0.0.0
appendonly yes
maxclients 10000
tcp-keepalive 60
protected-mode no
maxmemory 6gb
maxmemory-policy allkeys-lru
activedefrag yes


# 限制最大内存使用，防止系统 OOM
maxmemory 6gb
# 内存淘汰策略：优先淘汰最近最少使用的键
maxmemory-policy allkeys-lru
# 开启主动碎片整理（默认关闭，建议开启）
activedefrag yes

# Redis 7.x 的碎片整理参数（默认值即可，视情况可微调）
activedefrag-threshold-lower 10
activedefrag-threshold-upper 100
activedefrag-cycle-min 25
activedefrag-cycle-max 75

# 设置密码（强烈建议）
requirepass yourStrongPasswordHere

# 最大连接数（根据业务量设置）
maxclients 10000

# 保护模式，生产建议关闭（依赖防火墙而非本地网络）
protected-mode no

# 设置数据库数量（默认 16，实际用不到可设为 1 减少 overhead）
databases 1
```

# 五、测试

## 1.功能测试

```
redis-cli ping               # 应该返回 PONG
redis-cli set test_key 123   # 设置一个 key
redis-cli get test_key       # 应该返回 123
```

```
redis-cli --raw JSON.SET doc $ '{"name":"Alice","age":30}'
redis-cli --raw JSON.GET doc
redis-cli --raw JSON.DEL doc
```

或使用 `redisinsight` 工具图形化查看 JSON 结构。

## 2.性能测试

```
# 默认测试100000次请求，50并发，GET/SET
redis-benchmark -h <host> -p <port> -c 50 -n 100000

# 示例：测试 JSON 设置
redis-benchmark -t set,get -n 10000 -d 256
```

| 参数 | 含义                                |
| ---- | ----------------------------------- |
| `-n` | 请求总数（如 100000）               |
| `-c` | 并发连接数                          |
| `-d` | 每个 value 的数据大小（单位：字节） |
| `-t` | 测试命令（如 set、get、incr）       |
| `-q` | 安静模式，仅输出结果                |
| `-P` | 管道数量（批量请求）                |

## 3.高可用测试

```
# 人为停掉 master 容器/进程
docker stop redis-master

# Sentinel 日志应该有主从切换
kubectl logs redis-sentinel-0

# 验证新的 master 是否生效
redis-cli -p 26379 sentinel get-master-addr-by-name mymaster
```

## 4.内存/稳定性/模块压力测试

[memtier_benchmark](https://github.com/RedisLabs/memtier_benchmark)：支持更复杂的数据模型、JSON 结构、TLS 等

[redis-py 脚本测试](https://github.com/redis/redis-py)：你可用 Python 写脚本循环写入、删除、并发测试

# 六、编码

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
