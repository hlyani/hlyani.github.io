# Synology

# 1、安装

[教程](https://www.ainas.cc:88/?p=4331)

[教程2](https://mp.weixin.qq.com/s/j57qD181whsOKrQRH7paZw)

[DS918+ 7.2.1-69057 安装镜像](https://ainas.cc:5001/sharing/LECdoEJxC)

[操作系统pat文件](https://cndl.synology.cn/download/DSM/release/7.2.1/69057/DSM_DS918%2B_69057.pat)

[ChipGenius](https://ainas.cc:5001/fsdownload/hU0Oj1sKm/%E5%B7%A5%E5%85%B7%E9%9B%86%E5%90%88)

[群晖搜索助手](https://ainas.cc:5001/fsdownload/hU0Oj1sKm/%E5%B7%A5%E5%85%B7%E9%9B%86%E5%90%88)

## 1、安装镜像修改

通过Diskgenius修改安装镜像 grub.cfg

RPCB1(0)->boot->grub->grub.cfg

修改 vid、pid、sn、mac1、mac2

通过ChipGenius获取U盘vid、pid

## 2、制作U盘系统

Rufus

imageUSB

## 3、安装

进入Bios设置U盘为第一启动

进入启动，等待安装

通过群晖搜索助手 扫描synology IP地址

打开浏览器，IP:5000

安装 DiskStation Manager，选择 DSM_DS918+_69057.pat

根据步骤安装

## 4、使用m.2硬盘

[winhex](https://ainas.cc:5001/fsdownload/hU0Oj1sKm/%E5%B7%A5%E5%85%B7%E9%9B%86%E5%90%88)

[参考](https://wp.gxnas.com/10930.html)

> NVME硬盘群晖无法识别，因为群晖提亲设定了各个机型NVME所在的PCI位置，保存在/lib64/libsynonvme.so.1文件中。

##### 1、查看nvme信息

```
ls /dev/nvme*
```

```
udevadm info /dev/nvme0n1
P: /devices/pci0000:00/0000:00:1c.0/0000:01:00.0/nvme/nvme0/nvme0n1
```

> 需要使用上述  0000:00:1c.0，第三个字段是硬盘ID

##### 2、修改/lib64/libsynonvme.so.1

```
备份
cp /lib64/libsynonvme.so.1 /lib64/libsynonvme.so.1.bak
```

```
复制到windows上 使用winhex修改

使用winhex打开libsynonvme.so.1
ctrl+F 搜索 DS918+

在右侧找到原数据为0000:00:13.0和0000:00:13.1的字段，根据第一步查到的本机NVME所在的PCI位置，修改为0000:00:1d.0，顺便把另外一个nvme插槽也修改为0000:00:1d.1，修改后保存。
```

##### 3、上传到群晖，替换 /lib64/libsynonvme.so.1

```
chmod 755 /lib64/libsynonvme.so.1
```

##### 4、重启群晖。

##### 5、其他

```
fdisk -l /dev/nvme0n1
cat /proc/mdstat
分区
synopartition --part /dev/nvme0n1 7
mdadm --create /dev/md7 --level=1 --raid-devices=1 --force /dev/nvme0n1p3

格式化
mkfs.ext4 -F /dev/md7
mkfs.btrfs -f /dev/md7
```

# 参考

[工具集合](https://ainas.cc:5001/fsdownload/hU0Oj1sKm/%E5%B7%A5%E5%85%B7%E9%9B%86%E5%90%88)

[Tailscale](https://www.ainas.cc:88/?p=1184)
