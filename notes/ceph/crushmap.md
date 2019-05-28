# crushmap

```
# begin crush map
tunable choose_local_tries 0
tunable choose_local_fallback_tries 0
tunable choose_total_tries 50
tunable chooseleaf_descend_once 1
tunable chooseleaf_vary_r 1
tunable straw_calc_version 1

# devices
device 0 osd.0
device 1 osd.1
device 2 osd.2
device 3 osd.3
device 4 osd.4
device 5 osd.5
device 6 osd.6
device 7 osd.7
device 8 osd.8
device 9 osd.9
device 10 osd.10
device 11 osd.11
device 12 osd.12
device 13 osd.13
device 14 osd.14

# types
type 0 osd
type 1 host
type 2 chassis
type 3 rack
type 4 row
type 5 pdu
type 6 pod
type 7 room
type 8 datacenter
type 9 region
type 10 root

# buckets
host ceph-node1-sata {
	id -3		# do not change unnecessarily
	# weight 4.000
	alg straw
	hash 0	# rjenkins1
	item osd.0 weight 1.000
	item osd.1 weight 1.000
	item osd.2 weight 1.000
	item osd.3 weight 1.000
}
host ceph-node2-sata {
	id -4		# do not change unnecessarily
	# weight 4.000
	alg straw
	hash 0	# rjenkins1
	item osd.4 weight 1.000
	item osd.5 weight 1.000
	item osd.6 weight 1.000
	item osd.7 weight 1.000
}
host ceph-node3-sata {
	id -5		# do not change unnecessarily
	# weight 4.000
	alg straw
	hash 0	# rjenkins1
	item osd.8 weight 1.000
	item osd.9 weight 1.000
	item osd.10 weight 1.000
	item osd.11 weight 1.000
}
root sata {
	id -1		# do not change unnecessarily
	# weight 12.000
	alg straw
	hash 0	# rjenkins1
	item ceph-node1-sata weight 4.000
	item ceph-node2-sata weight 4.000
	item ceph-node3-sata weight 4.000
}
host ceph-node1-ssd {
	id -6		# do not change unnecessarily
	# weight 1.000
	alg straw
	hash 0	# rjenkins1
	item osd.12 weight 1.000
}
host ceph-node2-ssd {
	id -7		# do not change unnecessarily
	# weight 1.000
	alg straw
	hash 0	# rjenkins1
	item osd.13 weight 1.000
}
host ceph-node3-ssd {
	id -8		# do not change unnecessarily
	# weight 1.000
	alg straw
	hash 0	# rjenkins1
	item osd.14 weight 1.000
}
root ssd {
	id -2		# do not change unnecessarily
	# weight 3.000
	alg straw
	hash 0	# rjenkins1
	item ceph-node1-ssd weight 1.000
	item ceph-node2-ssd weight 1.000
	item ceph-node3-ssd weight 1.000
}

# rules
rule sata {
	ruleset 0
	type replicated
	min_size 1
	max_size 10
	step take sata
	step chooseleaf firstn 0 type host
	step emit
}
rule ssd {
	ruleset 1
	type replicated
	min_size 1
	max_size 10
	step take ssd
	step chooseleaf firstn 0 type host
	step emit
}

# end crush map
```

