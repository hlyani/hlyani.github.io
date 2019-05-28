# Ceph 调优

```

  cluster:                                                                                                                                                                                     
    id:     79d60ec1-cece-464e-9d2c-37ed3581a3dd                                                                                                                                               
    health: HEALTH_ERR                                                                                                                                                                         
            1 scrub errors                                                                                                                                                                     
            Possible data damage: 1 pg inconsistent


ceph health detail
ceph healthrepair 14.1a

#保留一段时间以来的访问记录，这样 Ceph 就能判断一客户端在一段时间内访问了某对象一次、还是多次（存活期与热度）。
#为缓存存储池保留的命中集数量
ceph osd pool set vms.cache hit_set_count 1

#为缓存存储池保留的命中集有效期
ceph osd pool set vms.cache hit_set_period 3600

#缓存存储池包含的脏对象达到多少比例时就把它们回写到后端的存储池
ceph osd pool set vms.cache cache_target_dirty_ratio 0.4

#缓存存储池内包含的已修改（脏的）对象达到此比例时，缓存层代理就会更快地把脏对象刷回到后端存储池
ceph osd pool set vms.cache cache_target_dirty_high_ratio 0.6

#缓存存储池包含的干净对象达到多少比例时，缓存代理就把它们赶出缓存存储池
ceph osd pool set vms.cache cache_target_full_ratio 0.8

#达到 max_bytes 阀值时 Ceph 就回写或赶出对象（200G）
ceph osd pool set vms.cache target_max_bytes 200000000000

#达到 max_objects 阀值时 Ceph 就回写或赶出对象(每个对象1M)
ceph osd pool set vms.cache target_max_objects 200000

#达到此时间（单位为秒）时，缓存代理就把某些对象从缓存存储池刷回到存储池
ceph osd pool set vms.cache cache_min_flush_age 600

#达到此时间（单位为秒）时，缓存代理就把某些对象从缓存存储池赶出
ceph osd pool set vms.cache cache_min_evict_age 1800

hit_set_count
描述:    为缓存存储池保留的命中集数量。此值越高， ceph-osd 守护进程消耗的内存越多。
类型:    整数
有效范围:    1. Agent doesn’t handle > 1 yet.

hit_set_period
描述:    为缓存存储池保留的命中集有效期。此值越高， ceph-osd 消耗的内存越多。
类型:    整数
实例:    3600 1hr

ceph osd pool set vms-cache hit_set_count 1
ceph osd pool set vms-cache hit_set_period 3600
ceph osd pool set vms-cache cache_target_dirty_ratio 0.4
ceph osd pool set vms-cache cache_target_dirty_high_ratio 0.6
ceph osd pool set vms-cache cache_target_full_ratio 0.8
ceph osd pool set vms-cache target_max_bytes 200000000000
ceph osd pool set vms-cache target_max_objects 200000
ceph osd pool set vms-cache cache_min_flush_age 600
ceph osd pool set vms-cache cache_min_evict_age 1800

ceph osd pool set images-cache hit_set_count 1
ceph osd pool set images-cache hit_set_period 3600
ceph osd pool set images-cache cache_target_dirty_ratio 0.4
ceph osd pool set images-cache cache_target_dirty_high_ratio 0.6
ceph osd pool set images-cache cache_target_full_ratio 0.8
ceph osd pool set images-cache target_max_bytes 200000000000
ceph osd pool set images-cache target_max_objects 200000
ceph osd pool set images-cache cache_min_flush_age 600
ceph osd pool set images-cache cache_min_evict_age 1800

ceph osd pool set volumes-cache hit_set_count 1
ceph osd pool set volumes-cache hit_set_period 3600
ceph osd pool set volumes-cache cache_target_dirty_ratio 0.4
ceph osd pool set volumes-cache cache_target_dirty_high_ratio 0.6
ceph osd pool set volumes-cache cache_target_full_ratio 0.8
ceph osd pool set volumes-cache target_max_bytes 200000000000
ceph osd pool set volumes-cache target_max_objects 200000
ceph osd pool set volumes-cache cache_min_flush_age 600
ceph osd pool set volumes-cache cache_min_evict_age 1800

ceph osd pool set backups-cache hit_set_count 1
ceph osd pool set backups-cache hit_set_period 3600
ceph osd pool set backups-cache cache_target_dirty_ratio 0.4
ceph osd pool set backups-cache cache_target_dirty_high_ratio 0.6
ceph osd pool set backups-cache cache_target_full_ratio 0.8
ceph osd pool set backups-cache target_max_bytes 200000000000
ceph osd pool set backups-cache target_max_objects 200000
ceph osd pool set backups-cache cache_min_flush_age 600
ceph osd pool set backups-cache cache_min_evict_age 1800

ceph osd pool set gnocchi-cache hit_set_count 1
ceph osd pool set gnocchi-cache hit_set_period 3600
ceph osd pool set gnocchi-cache cache_target_dirty_ratio 0.4
ceph osd pool set gnocchi-cache cache_target_dirty_high_ratio 0.6
ceph osd pool set gnocchi-cache cache_target_full_ratio 0.8
ceph osd pool set gnocchi-cache target_max_bytes 200000000000
ceph osd pool set gnocchi-cache target_max_objects 200000
ceph osd pool set gnocchi-cache cache_min_flush_age 600
ceph osd pool set gnocchi-cache cache_min_evict_age 1800
```