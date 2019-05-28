# ceph.conf

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

