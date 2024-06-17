# 网络命名空间 netns

## 1、基本命令

```
ip netns list
ip netns add test1
ip netns delete test1
ip netns exec <namespace id> ip a

ip link add veth-test1 type veth peer name veth-test2
ip link set veth-test1 netns test1
ip link set veth-test2 netns test2
ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1
ip netns exec test2 ip addr add 192.168.1.2/24 dev veth-test2
ip netns exec test1 ip link set dev veth-test1 up
ip netns exec test2 ip link set dev veth-test2 up
```

## 2、示例

```
# 创建test1,test2
ip netns add test1
ip netns add test2

# 开启test1,test2
ip netns exec test1 ip link set dev lo up
ip netns exec test2 ip link set dev lo up

# 创建一对veth
ip link add veth-test1 type veth peer name veth-test2

# 分配给test1，test2
ip link set veth-test1 netns test1
ip link set veth-test2 netns test2

# 给veth分配ip地址
ip netns exec test1 ip addr add 192.168.1.1/24 dev veth-test1
ip netns exec test2 ip addr add 192.168.1.2/24 dev veth-test2

# 启动veth
ip netns exec test1 ip link set dev veth-test1 up
ip netns exec test2 ip link set dev veth-test2 up

# result
# 在test1命名空间中可以ping通 192.168.1.2
# 在test2命名空间中可以ping通 192.168.1.1
ip netns exec test1 ping 192.168.1.2
ip netns exec test2 ping 192.168.1.1
```

