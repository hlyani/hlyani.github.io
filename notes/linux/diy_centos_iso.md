# CentOS iso 镜像定制

##### 1、下载并上传CentOS-7-x86_64-Minimal-1810.iso到虚拟机。

[centos下载地址](https://www.centos.org/download/)

##### 2、创建目录media，并挂载iso。

```
mkdir /media
mount CentOS-7-x86_64-Minimal-1810.iso /media
```

##### 3、创建镜像目录，并拷贝/media下的所有文件。

```
mkdir /root/TMPOS
cp -pRf /media/* /root/TMPOS/
cp /media/.discinfo /root/TMPOS/
cp /media/.treeinfo /root/TMPOS/
```

##### 4、修改centos-release（可选）。

[iso 系统定制](https://www.cnblogs.com/appresearch/p/5484450.html)

```
%define product_family CentOS

http://rpm.pbone.net/index.php3/stat/3/srodzaj/2/search/centos-release-7-3.1611.el7.centos.src.rpm
```

##### 5、修改centos-logos（可选）。

```
tar -xf centos-logos-70.0.6.tar.xz
tar -cvf centos-logos-70.0.6.tar.xz centos-logos-70.0.6/
cd centos-logos-70.0.6/pixmaps
```

##### 6、修改isolinux（可选）。

```
mkdir tmp
cp initrd.img tmp/
xz -dc initrd.img |cpio -id

vim .buildstamp
...
Product=CentOS
...

cd usr/share/pixmaps
find . |cpio -c -o |xz -9 --format=lzma > initrd.img
```

##### 7、修改LiveOS（可选）。

```
cd LiveOS/
unsquashfs squashfs.img
cd squashfs-root
mkdir t
mount -o rw rootfs.img t
cd usr/share/anaconda/pixmaps/
cd usr/share/anaconda/pixmaps/rnotes/en/
vim etc/os-release
vim etc/redhat-release
vim etc/system-release
vim etc/centos-release

vim usr/lib64/python2.7/site-packages/pyanaconda/ui/gui/hubs/progress.py
#self.set_warning(_("Use of this product is subject to the license agreement found at %s") % eulaLocation)
umoun t
rm -rf t
rm -rf squashfs.img
mksquashfs squashfs-root/ squashfs.img -all-root -noF
rm -rf squashfs-root
```

##### 8、准备ks.cfg文件 isolinux/ks.cfg。

```
安装前运行自定义脚本
%pre表示系统安装前，此时ISO镜像文件被挂载到内存中Linux的/mnt/source

安装后运行自定义脚本
%post 在系统安装后执行
–不带参数，其实就是在真实的操作系统里操作。
–nochroot 已安装的真实操作系统被挂载到内存虚拟操作系统中的/mnt/sysimage目录。这个参数的用途主要是配合%pre使用的。先将光盘里的文件copy到内存运行的虚拟操作系统，再从内存虚拟操作系统copy到已安装的真实操作系统操作。
%post --nochroot
mkdir /media
mount /dev/cdrom /media/
cp /media/test1.txt /mnt/sysimage/root/
%end
[说明]：上面命令实现了从ISO镜像中拷贝文本文件到安装好的真实操作系统中。
```

```ks.cfg
#version=DEVEL  
# Install OS instead of upgrade  
install  
# Keyboard layouts  
keyboard --vckeymap=us --xlayouts='us'
# Reboot after installation  
reboot  
# Root password  
rootpw 123456 
user --name="kyos" --password="123456" --groups=users 
# System timezone  
timezone Asia/Shanghai --isUtc --nontp
# System language
lang en_US.UTF-8
# Firewall configuration  
firewall --disabled
# SELinux configuration  
selinux --disabled
# System authorization information  
auth  --useshadow  --passalgo=sha512  
# Use graphical install  
#graphical  
text
firstboot --enable  
# SELinux configuration  
selinux --disabled  
# ignore unsupported hardware warning
unsupported_hardware
# SKIP CONFIGURING X
skipx
# Network information
network  --hostname=kyos
#Allow partition the system as needed  
%include /tmp/partition.ks  
  
%pre  
#!/bin/sh  
drives=""
for drv in `ls -1 /sys/block | grep "sd\|hd\|vd\|cciss"`; do
    if (grep -q 0 /sys/block/${drv}/removable); then
        d=`echo ${drv} | sed -e 's/!/\//'`
        drives="${drives} ${d}"
    fi
done
default_drive=`echo ${drives} | awk '{print $1}'`
act_mem=`cat /proc/meminfo | grep MemTotal | awk '{printf("%d",$2/1024)}'`  
echo "" > /tmp/partition.ks  
# System bootloader configuration  
echo "bootloader --location=mbr --driveorder=${default_drive}" >> /tmp/partition.ks
# Clear the Master Boot Record 
echo "zerombr" >> /tmp/partition.ks
# Partition clearing information 
echo "clearpart --all --drives=${default_drive}" >> /tmp/partition.ks  
echo "part /boot --fstype=xfs --ondisk=${default_drive} --asprimary --size=500" >> /tmp/partition.ks  
echo "part /boot/efi --fstype=efi --ondisk=${default_drive} --asprimary --size=500" >> /tmp/partition.ks
echo "part swap --fstype=swap --ondisk=${default_drive} --size=${act_mem}" >> /tmp/partition.ks  
echo "part / --fstype=xfs --grow --ondisk=${default_drive} --size=1" >> /tmp/partition.ks  

%end  

%packages 
@core
@kolla
%end  

%post --log=/var/log/post.log
#!/bin/bash

#change net device to ethx
#sed -i 's/quiet"/quiet net.ifnames=0 biosdevname=0"/g' /etc/default/grub
#grub2-mkconfig -o /boot/grub2/grub.cfg

cd /etc/sysconfig/network-scripts
before_network_script=$(ls ifcfg-*|grep -v lo|grep -v eth*)
net_dev_num=$(ls ifcfg-*|grep -v lo|wc -l)
rm -rf $before_network_script

for ((i=0;i<$net_dev_num;i++)); do

echo "TYPE=Ethernet
BOOTPROTO=static
DEVICE=eth$i
ONBOOT=no
DNS1=
IPADDR=
GATEWAY=
NETMASK=" |tee > /etc/sysconfig/network-scripts/ifcfg-eth$i

done

#add pcadmin to sudoers
echo "tmpos ALL=(ALL)       ALL" >> /etc/sudoers
# add tmpos to group users
usermod -a -G users tmpos

# kolla
systemctl disable postfix
systemctl disable NetworkManager
systemctl enable docker
systemctl enable ntpd.service

mkdir -p /etc/systemd/system/docker.service.d
tee /etc/systemd/system/docker.service.d/kolla.conf <<'EOF'
[Service]
MountFlags=shared
EOF

cp -r /usr/share/kolla-ansible/etc_examples/kolla /etc/kolla/
cp -r /usr/share/kystack /opt

%end
```

##### 9、修改isolinux.cfg文件,isolinux/isolinux.cfg。

```
找到lable linux，在append里面添加ks=cdrom:/isolinux/ks.cfg net.ifnames=0 biosdevname=0，修改LABLE=TMPOS（在之后构建TMPOS时-V参数会用到）。
net.ifnames=0 biosdevname=0 修改网卡名为ethx。

...
label linux
  menu label ^Install TMPOS
  kernel vmlinuz
# append initrd=initrd.img inst.stage2=hd:LABEL=CentOS\x207\x20x86_64 quiet
  append initrd=initrd.img ks=cdrom:/isolinux/ks.cfg inst.stage2=hd:LABEL=TMPOS quiet net.ifnames=0 biosdevname=0
...

[说明]：斜体部分可选，代表是否在安装时对网络部分进行提示交互。
其他相关label可以根据需求注释掉。
注意点：
1）memu label 后面的内容是在光盘引导起来菜单的内容，^后面的字母是菜单的快捷键；
2）通过inst.ks关键字指明ks.cfg文件位置；
3）inst.stages2标识的是系统按照介质位置，这里使用hd:LABEL表明寻找的是label为TMPOS的安装介质，使用LABEL关键字的好处是可以精确指定安装介质，为什么label是TMPOS，是因为我在制作光盘镜像的时候指定的，方法在后面有介绍。
```

```isolinux.cfg
default vesamenu.c32
timeout 10

display boot.msg

# Clear the screen when exiting the menu, instead of leaving the menu displayed.
# For vesamenu, this means the graphical background is still displayed without
# the menu itself for as long as the screen remains in graphics mode.
menu clear
menu background splash.png
menu title KYOS
menu vshift 8
menu rows 18
menu margin 8
#menu hidden
menu helpmsgrow 15
menu tabmsgrow 13

# Border Area
menu color border * #00000000 #00000000 none

# Selected item
menu color sel 0 #ffffffff #00000000 none

# Title bar
menu color title 0 #ff7ba3d0 #00000000 none

# Press [Tab] message
menu color tabmsg 0 #ff3a6496 #00000000 none

# Unselected menu item
menu color unsel 0 #84b8ffff #00000000 none

# Selected hotkey
menu color hotsel 0 #84b8ffff #00000000 none

# Unselected hotkey
menu color hotkey 0 #ffffffff #00000000 none

# Help text
menu color help 0 #ffffffff #00000000 none

# A scrollbar of some type? Not sure.
menu color scrollbar 0 #ffffffff #ff355594 none

# Timeout msg
menu color timeout 0 #ffffffff #00000000 none
menu color timeout_msg 0 #ffffffff #00000000 none

# Command prompt text
menu color cmdmark 0 #84b8ffff #00000000 none
menu color cmdline 0 #ffffffff #00000000 none

# Do not display the actual menu unless the user presses a key. All that is displayed is a timeout message.

menu tabmsg Press Enter to continue.

menu separator # insert an empty line
menu separator # insert an empty line

label linux
  menu label ^Install TMPOS
  menu default
  kernel vmlinuz
  append initrd=initrd.img inst.stage2=hd:LABEL=TMPOS quiet net.ifnames=0 biosdevname=0 inst.ks=hd:LABEL=TMPOS:/isolinux/ks.cfg

menu separator # insert an empty line
menu separator # insert an empty line

menu end
```

##### 10、安装相关软件。

```
yum -y install xorriso syslinux isomd5sum squashfs-tools createrepo
```

##### 11、准备efi启动。

```
mkdir -p /root/TMPOS/efi_image
dd bs=1M count=100 if=/dev/zero of=/root/TMPOS/efiboot.img
mkfs.vfat -n EFI /root/TMPOS/efiboot.img
umount -l /root/TMPOS/efi_image || true
mount /root/TMPOS/efiboot.img /root/TMPOS/efi_image

echo "default=0" >> /root/TMPOS/EFI/BOOT/BOOTX64.conf
echo "timeout 60" >> /root/TMPOS/EFI/BOOT/BOOTX64.conf
echo "hiddenmenu" >> /root/TMPOS/EFI/BOOT/BOOTX64.conf
echo "title Install TMPOS" >> /root/TMPOS/EFI/BOOT/BOOTX64.conf
echo "  kernel /vmlinuz biosdevname=0 showmenu=yes inst.stage2=hd:LABEL=TMPOS quiet inst.ks=hd:LABEL=TMPOS:/isolinux/ks.cfg" >> /root/TMPOS/EFI/BOOT/BOOTX64.conf
echo "  initrd /initrd.img" >> /root/TMPOS/EFI/BOOT/BOOTX64.conf

修改 /root/TMPOS/EFI/BOOT/grub.cfg
添加
...
menuentry 'Install TMPOS' --class fedora --class gnu-linux --class gnu --class os {
        linuxefi /images/pxeboot/vmlinuz inst.stage2=hd:LABEL=TMPOS quiet net.ifnames=0 biosdevname=0 inst.ks=hd:LABEL=TMPOS:/isolinux/ks.cfg
        initrdefi /images/pxeboot/initrd.img
...

cp -r /root/TMPOS/isolinux/vmlinuz /root/TMPOS/efi_image/
cp -r /root/TMPOS/isolinux/initrd.img /root/TMPOS/efi_image/
cp -rf /root/TMPOS/EFI/ /root/TMPOS/efi_image/

umount /root/TMPOS/efi_image
mv -f /root/TMPOS/efiboot.img /root/TMPOS/images/
rm -rf /root/TMPOS/efi_image/
```

##### 12、准备安装源。

```
 1）将需要的源追加复制到/root/TMPOS/Packages中。
 2) 清除/root/TMPOS/repodata 中除了*.comps.xml外的其他文件，并将新加的包名写入*.comps.xml里
      <packagereq type="default">ntp</packagereq>
      <packagereq type="default">vim-enhanced</packagereq>
      <packagereq type="default">iftop</packagereq>
      <packagereq type="default">telnet</packagereq>
      <packagereq type="default">nmap</packagereq>
      <packagereq type="default">mlocate</packagereq>
      <packagereq type="default">net-tools</packagereq>
      <packagereq type="default">atop</packagereq>
      <packagereq type="default">iftop</packagereq>
      <packagereq type="default">vim</packagereq>
      <packagereq type="default">wget</packagereq>
      <packagereq type="default">tmux</packagereq>
      <packagereq type="default">iotop</packagereq>
      <packagereq type="default">htop</packagereq>
      <packagereq type="default">expect</packagereq>
      <packagereq type="default">tree</packagereq>
      <packagereq type="default">python2-pip</packagereq>
      <packagereq type="default">epel-release</packagereq>
    </packagelist>
  </group>
  <group>
    <id>kolla</id>
    <name>Kolla</name>
    <name xml:lang="en_US">Kolla</name>
    <description>Kolla packages</description>
    <default>true</default>
    <uservisible>true</uservisible>
    <packagelist>
      <packagereq type="default">ansible</packagereq>
      <packagereq type="default">kolla-ansible</packagereq>
      <packagereq type="default">docker-ce</packagereq>
      <packagereq type="default">python-docker-py</packagereq>
      <packagereq type="default">python2-pbr</packagereq>
      <packagereq type="default">python2-oslo-utils</packagereq>
      <packagereq type="default">python2-oslo-config</packagereq>
      <packagereq type="default">python2-jinja2</packagereq>
      <packagereq type="default">openstack-selinux</packagereq>
      <packagereq type="default">python2-openstackclient</packagereq>
      <packagereq type="default">python2-magnumclient</packagereq>
    </packagelist>
  </group>
  
 3）构建源
 yum -y install createrepo
 cd /root/TMPOS
 createrepo -g repodata/*comps.xml .
 
 4) 在/root/TMPOS/isolinux/ks.cf中添加相应组名
 %packages
 @core
 @kolla
 %end
```

##### 13、构建TMPOS。

```
mkisofs参数简介
-o 指定映像文件的名称。
-o /root/boot.iso ：创建完后放在哪
-b指定在制作可开机光盘时所需的开机映像文件。
-c 制作可开机光盘时，会将开机映像文件中的no-eltorito-catalog全部内容作成一个文件。
-c isolinux/boot.cat ：指明引导时显示菜单的文件
-no-emul-boot 非模拟模式启动。
-boot-load-size 4 设置载入部分的数量。
-boot-info-table 在启动的图像中现实信息。
-joliet-long 使用 joliet 格式的目录与文件名称，长文件名支持。
-R 或 -rock 使用 Rock RidgeExtensions 。
-J 或 -joliet 使用 Joliet 格式的目录与文件名称。
-v 或 -verbose 执行时显示详细的信息。
-T 或-translation-table 建立文件名的转换表，适用于不支持 Rock Ridge Extensions 的系统上。
-V “CentOS 6.6 X86_64 boot disk”：指明光盘标签为”CentOS 6.6 X86_64 boot disk“
```

```
xorriso -as mkisofs \
	-V "TMPOS" -p "tmp" \
	-J -R \
	-graft-points \
	-b isolinux/isolinux.bin -no-emul-boot -boot-load-size 4 -boot-info-table \
	-isohybrid-mbr /usr/share/syslinux/isohdpfx.bin \
	-eltorito-alt-boot -e images/efiboot.img -no-emul-boot \
	-isohybrid-gpt-basdat \
	-o /root/TMPOS/TMPOS.iso /root/TMPOS/ \
	--boot-catalog-hide
	
implantisomd5 /root/TMPOS/TMPOS.iso
```

