# overlay2

# 一、查看镜像 id

```
 docker images |grep 3.9-alpine3.12
 
 python         3.9-alpine3.12        7254f7459375        3 months ago        44.2MB
```

# 二、查看镜像层

```
cat /var/lib/docker/image/overlay2/imagedb/content/sha256/7254f7459375ca357ad951c2a97d548455dc9bcc0064589c519ff7dc37708d7d|python3 -m json.tool
...
    "rootfs": {
        "type": "layers",
        "diff_ids": [
            "sha256:f4666769fca7a1db532e3de298ca87f7e3124f74d17e1937d1127cb17058fead",
            "sha256:f2bc6754fc86dfbd965dfb93127dc473c36bc44cf21f87a779dcd0f94d924f3d",
            "sha256:e25ee6f7192be198a37c00741fce057aaa0eef58b0fb77a9fc2a23dcd84a5302",
            "sha256:0d2c65b855c43c1c83cf24e866e70e232f3ba451aa6aa159adc1e4dc3d51f96a",
            "sha256:debde983e4fe3ad328c6aa3f37cf9ee30d268b8266e37da87417133d0cc252ad"
        ]
    }
...
```

```
docker inspect python:3.9-alpine3.12
...
        "RootFS": {
            "Type": "layers",
            "Layers": [
                "sha256:f4666769fca7a1db532e3de298ca87f7e3124f74d17e1937d1127cb17058fead",
                "sha256:f2bc6754fc86dfbd965dfb93127dc473c36bc44cf21f87a779dcd0f94d924f3d",
                "sha256:e25ee6f7192be198a37c00741fce057aaa0eef58b0fb77a9fc2a23dcd84a5302",
                "sha256:0d2c65b855c43c1c83cf24e866e70e232f3ba451aa6aa159adc1e4dc3d51f96a",
                "sha256:debde983e4fe3ad328c6aa3f37cf9ee30d268b8266e37da87417133d0cc252ad"
            ]
        },
...
```

# 三、查看元数据信息及 rootfs

## 1、第一层

```
ls /var/lib/docker/image/overlay2/layerdb/sha256/f4666769fca7a1db532e3de298ca87f7e3124f74d17e1937d1127cb17058fead/
cache-id  diff  size  tar-split.json.gz

cat cache-id
a9410091d624483c6ea184a139807000b39e9ee2d4e6fda29ecbfb7d7a914856

ls /var/lib/docker/overlay2/a9410091d624483c6ea184a139807000b39e9ee2d4e6fda29ecbfb7d7a914856
committed  diff  link

ls /var/lib/docker/overlay2/a9410091d624483c6ea184a139807000b39e9ee2d4e6fda29ecbfb7d7a914856/diff/
bin  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

cat link
ROYFFPZS3DDRNW5YDCZ4RC6TKS

ls /var/lib/docker/overlay2/l/ROYFFPZS3DDRNW5YDCZ4RC6TKS/
bin  dev  etc  home  lib  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

## 2、第二层

```
echo -n "sha256:f4666769fca7a1db532e3de298ca87f7e3124f74d17e1937d1127cb17058fead sha256:f2bc6754fc86dfbd965dfb93127dc473c36bc44cf21f87a779dcd0f94d924f3d"|sha256sum

25370907defafe1eacf55dd1df121dd16890773e4a3d0637629de56ed6c69198

ls /var/lib/docker/image/overlay2/layerdb/sha256/25370907defafe1eacf55dd1df121dd16890773e4a3d0637629de56ed6c69198/
cache-id  diff  parent  size  tar-split.json.gz
```

## 3、第三层

```
echo -n "sha256:25370907defafe1eacf55dd1df121dd16890773e4a3d0637629de56ed6c69198 sha256:e25ee6f7192be198a37c00741fce057aaa0eef58b0fb77a9fc2a23dcd84a5302"|sha256sum

2d87c3c42dcce905c4f0aafd9657cc0247a9a79c461f0e8abf81a2c8523dacfb
```

## 4、第四层

```
echo -n "sha256:2d87c3c42dcce905c4f0aafd9657cc0247a9a79c461f0e8abf81a2c8523dacfb sha256:0d2c65b855c43c1c83cf24e866e70e232f3ba451aa6aa159adc1e4dc3d51f96a"|sha256sum

49c9ddedef77ac89647c786c3bd53c7c6bb9db897a0465087e1847afeb5c9eba
```

## 5、第五层

```
echo -n "sha256:49c9ddedef77ac89647c786c3bd53c7c6bb9db897a0465087e1847afeb5c9eba sha256:debde983e4fe3ad328c6aa3f37cf9ee30d268b8266e37da87417133d0cc252ad"|sha256sum

66604bb690d7ccdba67800376f20617e4821baf9c716534a399288bd9845be12
```

# 三、查看挂载信息

```
docker run -it --rm python:3.9-alpine3.12 sh

mount

rootfs on / type rootfs (rw)
overlay on / type overlay (rw,relatime,lowerdir=/var/lib/docker/overlay2/l/WEJADLTSDEUAABOXXY6UZFIO4R:/var/lib/docker/overlay2/l/5Q5UEBNKJEEDE6YZ265HDXGXXZ:/var/lib/docker/overlay2/l/GRABQF7VZXYCVN52TF2YEHMO27:/var/lib/docker/overlay2/l/EQ2OTBRW4K7MN6Z2R4QKIIOFHR:/var/lib/docker/overlay2/l/4X5YCB5KOIFOQZEK5P6GAZJY7E:/var/lib/docker/overlay2/l/ROYFFPZS3DDRNW5YDCZ4RC6TKS,upperdir=/var/lib/docker/overlay2/6442daf60a7587396f17c727e570c54538e5731bdfd50af4cb2ac2e59a0e2382/diff,workdir=/var/lib/docker/overlay2/6442daf60a7587396f17c727e570c54538e5731bdfd50af4cb2ac2e59a0e2382/work)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
tmpfs on /dev type tmpfs (rw,nosuid,size=65536k,mode=755)
devpts on /dev/pts type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
sysfs on /sys type sysfs (ro,nosuid,nodev,noexec,relatime)
tmpfs on /sys/fs/cgroup type tmpfs (ro,nosuid,nodev,noexec,relatime,mode=755)
cgroup on /sys/fs/cgroup/systemd type cgroup (ro,nosuid,nodev,noexec,relatime,xattr,release_agent=/usr/lib/systemd/systemd-cgroups-agent,name=systemd)
cgroup on /sys/fs/cgroup/devices type cgroup (ro,nosuid,nodev,noexec,relatime,devices)
cgroup on /sys/fs/cgroup/perf_event type cgroup (ro,nosuid,nodev,noexec,relatime,perf_event)
cgroup on /sys/fs/cgroup/net_cls,net_prio type cgroup (ro,nosuid,nodev,noexec,relatime,net_prio,net_cls)
cgroup on /sys/fs/cgroup/cpuset type cgroup (ro,nosuid,nodev,noexec,relatime,cpuset)
cgroup on /sys/fs/cgroup/cpu,cpuacct type cgroup (ro,nosuid,nodev,noexec,relatime,cpuacct,cpu)
cgroup on /sys/fs/cgroup/freezer type cgroup (ro,nosuid,nodev,noexec,relatime,freezer)
cgroup on /sys/fs/cgroup/hugetlb type cgroup (ro,nosuid,nodev,noexec,relatime,hugetlb)
cgroup on /sys/fs/cgroup/pids type cgroup (ro,nosuid,nodev,noexec,relatime,pids)
cgroup on /sys/fs/cgroup/memory type cgroup (ro,nosuid,nodev,noexec,relatime,memory)
cgroup on /sys/fs/cgroup/blkio type cgroup (ro,nosuid,nodev,noexec,relatime,blkio)
mqueue on /dev/mqueue type mqueue (rw,nosuid,nodev,noexec,relatime)
shm on /dev/shm type tmpfs (rw,nosuid,nodev,noexec,relatime,size=65536k)
/dev/mapper/centos-root on /etc/resolv.conf type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/centos-root on /etc/hostname type xfs (rw,relatime,attr2,inode64,noquota)
/dev/mapper/centos-root on /etc/hosts type xfs (rw,relatime,attr2,inode64,noquota)
devpts on /dev/console type devpts (rw,nosuid,noexec,relatime,gid=5,mode=620,ptmxmode=666)
proc on /proc/bus type proc (ro,relatime)
proc on /proc/fs type proc (ro,relatime)
proc on /proc/irq type proc (ro,relatime)
proc on /proc/sys type proc (ro,relatime)
proc on /proc/sysrq-trigger type proc (ro,relatime)
tmpfs on /proc/acpi type tmpfs (ro,relatime)
tmpfs on /proc/kcore type tmpfs (rw,nosuid,size=65536k,mode=755)
tmpfs on /proc/keys type tmpfs (rw,nosuid,size=65536k,mode=755)
tmpfs on /proc/timer_list type tmpfs (rw,nosuid,size=65536k,mode=755)
tmpfs on /proc/timer_stats type tmpfs (rw,nosuid,size=65536k,mode=755)
tmpfs on /proc/sched_debug type tmpfs (rw,nosuid,size=65536k,mode=755)
tmpfs on /proc/scsi type tmpfs (ro,relatime)
tmpfs on /sys/firmware type tmpfs (ro,relatime)
```

