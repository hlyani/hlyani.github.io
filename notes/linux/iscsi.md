# iscsi 相关


## 一、服务端
##### 1、安装
```
yum -y install targetcli
systemctl start target
systemctl enable target
```
##### 2、Create a Backstore
```
targetcli
/backstores/fileio create file1 /tmp/disk1.img 200M write_back=false
/backstores/block create name=block_backend dev=/dev/sdb
/backstores/pscsi/ create name=pscsi_backend dev=/dev/sr0
/backstores/ramdisk/ create name=rd_backend size=1GB
```
##### 3、Create an iSCSI Target
```
targetcli
iscsi/
create //create iqn.2006-04.com.example:444
```
##### 4、Configure an iSCSI Portal
```
iqn.2006-04.example:444/tpg1/
portals/ create
portals/ create 192.168.122.137
```
##### 5、 Configure ACLs
```
/iscsi/iqn.20...mple:444/tpg1> acls/
create iqn.2006-04.com.example.foo:888
```
##### 6、Configure LUNs
```
luns/ create /backstores/ramdisk/rd_backend 
/iscsi/iqn.2016-02.local.itzgeek.server:disk1/tpg1/
set attribute generate_node_acls=1
set attribute authentication=0
clearconfig confirm=True
```
## 二、客户端
##### 1、安装客户端（在需要连接iscsi的主机中操作）
```
yum -y install iscsi-initiator-utils
```
##### 2、设置授权客户端的iqn，编辑/etc/iscsi/initiatorname.iscsi要和服务端（提供iscsi的服务器）acls配置一致
```
vim /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2003-01.org.linux-iscsi.hl.x8664:sn.18da323b85e7
systemctl restart iscsid
```
##### 3、发现并登陆iscsi服务器-t senbtargets”表示发布的target（我这上面缩写为“-t st”），选项“-p ”ip：port  用来指定服务器IP地址。“-m node”选项表示管理目标节点，选项“-l”表示登录连接（--login 也可以）
```
iscsiadm -m discovery -t st -p 192.168.21.33 -l
lsblk
blkid
iscsiadm -m node -L all
iscsiadm -m node -U all
Optionally, set auto_cd_after_create to false to prevent targetcli from automatically changing object contexts after the creation of a new object:
set global auto_add_mapped_luns=false
```
##### 4、设置开机自动登录
```
sudo iscsiadm -m node -o update -n node.startup -v automatic 
```