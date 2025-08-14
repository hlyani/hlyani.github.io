# ceph.conf

# 一、优化

## 1、 Kernel pid max

```
echo 4194303 > /proc/sys/kernel/pid_max
```

## 2、 设置MTU，交换机端需要支持该功能，系统网卡设置才有效果

```
配置文件追加MTU=9000
```

## 3、 read_ahead, 通过数据预读并且记载到随机访问内存方式提高磁盘读操作

```
echo “8192” > /sys/block/sda/queue/read_ahead_kb
```

## 4、 swappiness, 主要控制系统对swap的使用

```
echo “vm.swappiness = 0″/etc/sysctl.conf ; sysctl –p
```

## 5、 I/O Scheduler，SSD要用noop，SATA/SAS使用deadline

```
echo “deadline” >/sys/block/sd[x]/queue/scheduler
echo “noop” >/sys/block/sd[x]/queue/scheduler
```

# 二、配置

```
[global]
fsid = 6c4d5e9e-03ee-4812-a565-cec78596ae68
mon_initial_members = comp-1, comp-2, comp-3, comp-4, comp-5
mon_host = 172.18.20.1,172.18.20.2,172.18.20.3,172.18.20.4,172.18.20.5
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx
osd pool default size = 2
osd pool default min size = 1
mon_pg_warn_max_per_osd = 0
mon osd full ratio = .98
mon osd nearfull ratio = .95

[osd]
osd journal size = 20000
osd mkfs type = xfs
osd mkfs options xfs = -f

filestore xattr use omap = true
filestore min sync interval = 10
filestore max sync interval = 15
filestore queue max ops = 25000
filestore queue max bytes = 10485760
filestore queue committing max ops = 5000
filestore queue committing max bytes = 10485760000

journal max write bytes = 1073714824
journal max write entries = 10000
journal queue max ops = 50000
journal queue max bytes = 10485760000

osd max write size = 512
osd client message size cap = 2147483648
osd deep scrub stride = 131072
osd op threads = 8
osd disk threads = 4
osd map cache size = 1024
osd map cache bl size = 128
osd mount options xfs = "rw,noexec,nodev,noatime,nodiratime,nobarrier"
osd recovery op priority = 4
osd recovery max active = 10
osd max backfills = 4
osd crush update on start = false

[client]
rbd cache = true
rbd cache size = 268435456
rbd cache max dirty = 134217728
rbd cache max dirty age = 5

[client.glance]
	key = AQA5G6JXXgc7KBAAh6ug4Amkb0nMdS7BY4IUmQ==

[client.cinder]
	key = AQBLHKJXNZe6MhAA6QBlPu7tjFYQPBhRkQaRnA==

[client.cinder-backup]
	key = AQDzxaFXbd5jABAARm71lJW1anfe6XMFxuGiAw==
```

```
[global]
cluster_network = 192.168.180.0/24         #集群网络
public_network = 192.168.170.0/24          #管理网络，分配集群的外网网段，即对外数据交流的网段。
osd_pool_default_size = 2                  #osd的默认副本数，默认值3
osd_pool_default_min_size = 1              # 这是处于degraded状态的副本数目，它应该小于osd_pool_default_size的值，
										     为存储池中的object设置最小副本数目来确认写操作。即使集群处于degraded状态。
										     如果最小值不匹配，Ceph将不会确认写操作给客户端。
osd_pool_default_pg_num = 128              #每个存储池默认的pg数，默认32
osd_pool_default_pgp_num = 128             #PG和PGP的个数应该保持一致。PG和PGP的值很大程度上取决于集群大小，默认32
osd_crush_chooseleaf type = 1              #CRUSH规则用到chooseleaf时的bucket的类型，默认值就是1
osd_journal_size = 1024                    # 缺省值为0。你应该使用这个参数来设置日志大小。
                                             日志大小应该至少是预期磁盘速度和filestore最大同步时间间隔的两倍。如果使用了SSD日志，
                                             最好创建大于10GB的日志，并调大filestore的最小、最大同步时间间隔。
mon_pg_warn_max_per_osd = 1000             #每个osd上pg数量警告值，这个可以根据具体规划来设定
mon_osd_full_ratio = .85                   #存储使用率达到85%将不再提供数据存储，默认0.95
mon_osd_backfillfull_ratio = .80           #OSD被认为已满而不能回填之前的设备空间利用率阈值，默认0.90
mon_osd_nearfull_ratio = .75               #存储使用率达到70%集群将会warn状态，默认0.85
osd_deep_scrub_randomize_ratio = 0.01      #随机深度清洗概率,值越大，随机深度清洗概率越高,太高会影响业务

[mon]
mon_clock_drift_allowed = 1                #monitor间的clock drift，默认值0.05
mon_osd_min_down_reporters = 13            #向monitor报告down的最小OSD数，默认值2
mon_osd_down_out_interval = 600       	   #标记一个OSD状态为down和out之前ceph等待的秒数，默认值600
mon_cpu_threads = 4
mon_osd_cache_size = 500
mon_osd_cache_size_min = 134217728

[osd]
osd_journal_size = 20000                   #osd journal大小，默认5120   
osd_max_write_size = 512                   #OSD一次可写入的最大值(MB)，默认值90   
osd_client_message_size_cap = 2147483648   #客户端允许在内存中的最大数据(bytes)，默认值524288000,500M
osd_deep_scrub_stride = 131072             #在Deep Scrub时候允许读取的字节数(bytes)，默认值524288                         
osd_map_cache_size = 1024                  #保留OSD Map的缓存(MB)，默认值50
osd_recovery_sleep = 0                     #recovery的时间间隔，会影响recovery时长，如果recovery导致业务不正常，可以调大该值，增加时间间隔，默认0
osd_recovery_op_priority = 3               #恢复操作优先级，取值1-63，值越高占用资源越高，默认值3
osd_recovery_max_chunk = 1048576           #设置恢复数据块的大小，以防网络阻塞，默认为8388608
osd_recovery_max_active = 0                #同一时间内活跃的恢复请求数，默认值0
osd_recovery_max_single_start = 1          # 和osd_recovery_max_active一起使用。
                                             假设配置osd_recovery_max_single_start为1，
                                             osd_recovery_max_active为3
                                             意味着OSD在某个时刻会为一个PG启动一个恢复操作，而且最多可以有三个恢复操作同时处于活动状态。
osd_max_backfills = 4                      #一个OSD允许的最大backfills数，默认值1 
osd_min_pg_log_entries = 250               #修建PGLog是保留的最小PGLog数，默认值250 
osd_max_pg_log_entries = 10000             #修建PGLog是保留的最大PGLog数，默认值10000 
osd_mon_heartbeat_interval = 40            #OSD ping一个monitor的时间间隔，默认值30 
ms_dispatch_throttle_bytes = 1048576000    #等待派遣的最大消息数，默认值 104857600
objecter_inflight_ops = 819200             #客户端流控，允许的最大未发送io请求数，超过阀值会堵塞应用io，为0表示不受限，默认值1024
osd_op_log_threshold = 50                  #一次显示多少操作的log，默认值5 
filestore_min_sync_interval = 0.1          #从日志到数据盘最小同步间隔(seconds)，默认值0.01
filestore_max_sync_interval = 15           #从日志到数据盘最大同步间隔(seconds)，默认5
filestore_queue_max_ops = 25000            #数据盘最大接受的操作数，默认值50
filestore_queue_max_bytes = 1048576000     #数据盘一次操作最大字节数(bytes)，默认104857600
filestore_split_multiple = 8               #前一个子目录分裂成子目录中的文件的最大数量，默认值2   
filestore_merge_threshold = -10            #前一个子类目录中的文件合并到父类的最小数量，默认值-10
filestore_fd_cache_size = 1024             #对象文件句柄缓存大小，默认值128
filestore_op_threads = 32                  #并发文件系统操作数，默认值2
journal_max_write_bytes = 1073714824       #journal一次性写入的最大字节数(bytes)，默认值10485760
journal_max_write_entries = 10000          #journal一次性写入的最大记录数，默认值100  
osd_scrub_begin_hour = 22                  #清洗开始时间为晚上22点，默认0
osd_scrub_end_hour = 7                     #清洗结束时间为早上7点，默认0
osd_crush_update_on_start = false          # 新加的osd会up/in,但并不会更新crushmap，prepare+active期间不会导致数据迁移，默认true
osd_crush_chooseleaf_type = 0              #CRUSH规则用到chooseleaf时的bucket的类型，默认值1 
rbd_op_threads = 10                        #rbd 操作线程数，默认值1

[client]
rbd_cache = true                           #RBD缓存，默认值 true
rbd_op_threads = 10                        #rbd 操作线程数，默认值1
rbd_cache_size = 335544320                 #RBD缓存大小(bytes)，默认值33554432
rbd_cache_max_dirty = 134217728            #缓存为write-back时允许的最大dirty字节数(bytes)，如果为0，使用write-through，默认值25165824
rbd_cache_max_dirty_age = 30               #在被刷新到存储盘前dirty数据存在缓存的时间(seconds)，默认值1
rbd_cache_writethrough_until_flush = false # 该选项是为了兼容linux-2.6.32之前的virtio驱动，
                                             避免因为不发送flush请求，数据不回写，设置该参数后，
                                             librbd会以writethrough的方式执行io，
                                             直到收到第一个flush请求，才切换为writeback方式。默认值true
rbd_cache_max_dirty_object = 2             # 最大的Object对象数，表示通过rbd cache size计算得到，
                                             librbd默认以4MB为单位对磁盘Image进行逻辑切分，
                                             每个chunk对象抽象为一个Object，librbd中以Object为单位来管理缓存，
                                             增大该值可以提升性能，默认值0
rbd_cache_target_dirty = 235544320         #开始执行回写过程的脏数据大小，不能超过 rbd_cache_max_dirty，默认值16777216
```

# 三、其他

```
ceph osd set-nearfull-ratio 0.85
ceph osd set-backfillfull-ratio 0.90
ceph osd set-full-ratio 0.95
```

