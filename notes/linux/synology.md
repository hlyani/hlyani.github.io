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

# 参考

[工具集合](https://ainas.cc:5001/fsdownload/hU0Oj1sKm/%E5%B7%A5%E5%85%B7%E9%9B%86%E5%90%88)

[Tailscale](https://www.ainas.cc:88/?p=1184)
