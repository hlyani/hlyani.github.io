# Redis

# 一、高可用部署

[redis-stack-sentinel](https://github.com/hlyani/redis-stack-sentinel)

```
SENTINEL get-master-addr-by-name mymaster
SENTINEL masters
SENTINEL replicas mymaster
SENTINEL FAILOVER mymaster
SENTINEL slaves mymaster
INFO SENTINEL
ACL LIST
```

普通认证 √

```
# redis
requirepass qwe
masterauth qwe

# sentinel
sentinel auth-user mymaster default
sentinel auth-pass mymaster qwe
```

~~acl 认证~~ × 未验证通过

```
# redis
user sentinel on >{{ .Values.auth.password }} +client +subscribe +publish +ping +info +multi +slaveof +config +exec

# sentinel
sentinel auth-user mymaster sentinel
sentinel auth-pass mymaster qwe
```

# 二、基础使用

连接

```
redis-cli -h 127.0.0.1 -p 6379 --user redis -a infini_rag_flow_helm ping
```

```
REDISCLI_AUTH=qwe redis-cli --user default -p 6379 ping
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
| FT._LIST                       | 查看索引                                               |
| FT.INFO checkpoints            | 可查看每个索引的详细结构与配置                         |

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

# 四、其他

```
# 直接搜索 checkpoints 索引中 @thread_id 为 1943868169984831490 的文档，返回全部字段和匹配结果。
FT.SEARCH checkpoints "@thread_id:{1943868169984831490}"
# 执行搜索但不返回任何文档，只返回匹配的文档数量
FT.SEARCH checkpoints "@thread_id:{1943840008152707073}" LIMIT 0 0
# 搜索并仅返回 1 个匹配的文档（只返回 document id，不返回字段内容）
FT.SEARCH checkpoints "@thread_id:{1943840008152707073}" LIMIT 0 1 RETURN 0
```

```
import redis

# 连接 Redis
r = redis.Redis(host='localhost', port=6379, decode_responses=True)

# 执行 FT.SEARCH 查询（thread_id 是 TAG 类型，需要加 {}）
query = '@thread_id:{1943868169984831490}'
result = r.execute_command('FT.SEARCH', 'checkpoints', query)
```

```
# 模糊查询 json key 所占用内存和每个key 占用内存
redis-cli --scan --pattern "checkpoint:*" | while read key; do
  redis-cli MEMORY USAGE "$key"
done | awk '
  {sum += $1; count += 1}
  END {
    printf "Total Keys: %d\n", count;
    printf "Total Memory: %.2f MB\n", sum / 1024 / 1024;
    printf "Average per Key: %.2f KB\n", sum / count / 1024;
  }'
```

# 五、优化

[redis配置](https://github.com/redis/redis/blob/7.4.4/redis.conf)

```
# Redis 监听端口
port 6379

dir /tmp

# 监听所有网卡（用于 Kubernetes 或外部访问）
bind 0.0.0.0

# 启用 AOF 持久化机制（适合数据安全性要求高的业务）
appendonly yes

appendfsync no
no-appendfsync-on-rewrite yes

# 设置最大客户端连接数，防止连接耗尽资源
maxclients 10000

# 启用 TCP keepalive，单位为秒（建议开启以发现死连接）
tcp-keepalive 60

tcp-backlog 10240

timeout 300

hz 100

# 禁用 protected mode，适用于内网或 Kubernetes 环境
protected-mode no

# 限制 Redis 最大内存，防止系统 OOM（单位可为 kb, mb, gb）
maxmemory 8gb

# 内存淘汰策略：从所有键中优先淘汰最少使用的键（LRU）
maxmemory-policy allkeys-lru

# 启用主动碎片整理（默认关闭，建议开启以优化长期运行性能）
activedefrag yes

# Redis 7.x 的碎片整理参数（默认值即可，如需调优可参考以下）
# 当碎片率超过该值时触发整理
active-defrag-threshold-lower 10
# 达到更高碎片率后提升整理频率
active-defrag-threshold-upper 100
# 最小整理周期占 CPU 百分比
active-defrag-cycle-min 25
# 最大整理周期占 CPU 百分比
active-defrag-cycle-max 75

repl-backlog-size 128mb
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit pubsub 32mb 8mb 60

# 设置密码（强烈建议设置强密码以保障安全）
requirepass yourStrongPasswordHere

# 设置数据库数量（默认 16；如业务只需一个，可设为 1 降低开销）
databases 1

# =========================
# 🧑 ACL 用户权限配置
# =========================

# 默认用户：允许所有命令和键
user default on >strongDefaultPass allcommands allkeys

# 只读用户（readonly）：只能读不能写
user readonly on >readonlyPass +@read -@write ~*

# 只写用户（writer）：只能写不能读
user writer on >writerPass +@write -@read ~*


# 主节点设置访问密码
requirepass yourStrongPasswordHere

# 从节点设置 masterauth，用于向主节点认证
masterauth yourStrongPasswordHere

# 从节点向主节点请求完整的 RDB 快照（全量同步）；
# 然后持续执行命令传播（增量同步）以保持数据一致性。
replicaof redis-0.redis-headless 6379
```

> ACL SETUSER default on >yourStrongPassword allcommands allkeys

[sentinel配置](https://github.com/redis/redis/blob/7.4.4/sentinel.conf)

```
port 26379
dir /tmp

# Sentinel 监控主节点（mymaster 是主集群名称）
sentinel monitor mymaster redis-0.redis-headless 6379 2

# 设置主节点密码（和 Redis 的 requirepass 保持一致）
# 设置用于连接主/从 Redis 的密码
sentinel auth-pass mymaster {{ .Values.redis.password }}

# 判断主节点“故障”需要 5 秒没有回应
sentinel down-after-milliseconds mymaster 6000

# 故障转移最多等待 10 秒
sentinel failover-timeout mymaster 10000

# 故障转移时最多同时对几个从节点同步新主节点数据
sentinel parallel-syncs mymaster 1

# 防止错误选主（主从未完全同步前不选）
sentinel deny-scripts-reconfig yes

# 控制 Redis 实例在 无密码保护时是否限制访问。
protected-mode no

# 启用后，Redis Sentinel 将动态解析主节点或从节点的主机名（DNS 名称），而不是只解析一次并缓存 IP 地址。
sentinel resolve-hostnames yes
```

# 六、测试

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

redis-cli -p 26379 sentinel masters
```

## 4.内存/稳定性/模块压力测试

[memtier_benchmark](https://github.com/RedisLabs/memtier_benchmark)：支持更复杂的数据模型、JSON 结构、TLS 等

[redis-py 脚本测试](https://github.com/redis/redis-py)：你可用 Python 写脚本循环写入、删除、并发测试

# 七、编码

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

哨兵连接

```
redis:
  enable: true
  host: 127.0.0.1
  sentinel:
    - redis-sentinel-0.redis-sentinel-headless.default.svc.cluster.local
    - redis-sentinel-1.redis-sentinel-headless.default.svc.cluster.local
    - redis-sentinel-2.redis-sentinel-headless.default.svc.cluster.local
  port: 26379
  db: 0
  masterSet: mymaster
  max_connections: 1000
  default_ttl: 60
  socket_timeout: 3
  socket_keepalive: true
  health_check_interval: 30
  refresh_on_read: true
  retry_on_timeout: true


sentinels = [
    (host, port) for host in sentinel
]
sentinel_client = Sentinel(
    sentinels,
    socket_timeout=socket_timeout,
    retry_on_timeout=retry_on_timeout,
    health_check_interval=health_check_interval
)
redis_client = sentinel_client.master_for(master)
response = await redis_client.ping()
```

