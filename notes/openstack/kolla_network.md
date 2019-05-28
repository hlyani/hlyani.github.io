# kolla网络规划

```
kolla_internal_vip_address: "192.168.110.10"
kolla_external_vip_address: "192.168.21.10"
network_interface: "em1"
kolla_external_vip_interface: "em2"
#api_interface: "{{ network_interface }}"
#storage_interface: "{{ network_interface }}"
#cluster_interface: "{{ network_interface }}"
#tunnel_interface: "{{ network_interface }}"
#dns_interface: "{{ network_interface }}"
neutron_external_interface: "em3"

em1：管理网网卡（internel、存储。。。）
em2：管理网外部网卡（public）
em3：外网网卡（浮动ip等，up网卡，不用配置ip）
```