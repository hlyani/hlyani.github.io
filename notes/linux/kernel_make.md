# kernel 内核编译

# 一、构建交叉编译容器

> x86_64_aarch64

Dockerfile

```
From ubuntu:22.04

ARG http_proxy=http://192.168.0.55:1080
ARG https_proxy=$http_proxy
ARG no_proxy=$no_proxy
ENV http_proxy=$http_proxy
ENV https_proxy=$https_proxy
ENV no_proxy=$no_proxy

ENV PATH=$PATH:/opt/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/lib

WORKDIR /opt

RUN sed -i s@/archive.ubuntu.com/@/mirrors.ustc.edu.cn/@g /etc/apt/sources.list && \
    apt update && \
    DEBIAN_FRONTEND=noninteractive apt install --no-install-recommends --no-install-suggests -y dracut git make libncurses-dev libelf-dev bison flex libssl-dev bc u-boot-tools vim wget xz-utils && \
    rm -rf /var/lib/apt/lists/* && \
    ln -s /usr/bin/python3 /usr/bin/python && \
    wget --no-check-certificate https://releases.linaro.org/components/toolchain/binaries/latest-7/aarch64-linux-gnu/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz && \
    tar -xf gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz && \
    rm -rf gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz && \
    alias make='make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu-'
```

```
docker build -t gcc-linaro-7.5.0:1.0.0 .
```

# 二、容器内编译内核

## 1、直接编译

```
docker run -it --rm --privileged -v $PWD/../:/kernel -w /kernel /gcc-linaro-7.5.0:1.0.0 /bin/sh -c "sh << EOF
make config
make oldconfig
make
EOF"
```

## 2、编译boot.img

```
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- rockchip_linux_defconfig
cp arch/arm64/configs/rockchip_linux_defconfig .config
make menuconfig
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- rk3399-firefly-linux.img -j $(nproc)
mkdir -p ../out/
cp boot.img ../out/
```

## 3、检查内核是否支持容器相关功能

```
wget https://raw.githubusercontent.com/moby/moby/master/contrib/check-config.sh
chmod +x check-config.sh
./check-config.sh .config
```

# 三、安装依赖

```
yum install -y make ncurses-devel elfutils-libelf-devel bison flex openssl-devel bc gcc-c++ gcc
```

## 1、gcc 源码编译

```
wget https://mirrors.ustc.edu.cn/gnu/gcc/gcc-7.3.0/gcc-7.3.0.tar.gz

mkdir -p /usr/local/gcc
tar -xvf /hl/gcc-7.3.0.tar.gz -C /usr/local/gcc/
cd /usr/local/gcc/gcc-7.3.0/
./contrib/download_prerequisites 
HTTPS_PROXY=192.168.0.103:1080 ./contrib/download_prerequisites 
mkdir gcc-build-7.3.0
cd gcc-build-7.3.0/
yum install -y gmp-devel mpfr-devel libmpc-devel
../configure --enable-languages=c,c++ --disable-multilib
make -j8
make install -j8

rpm -q gcc
rpm -q gcc-c++
rpm -e gcc-4.8.5-28.el7_5.1.aarch64
rpm -e gcc-c++-4.8.5-28.el7_5.1.aarch64
```

## 2、下载源码

```
yumdownloader --source kernel
```

```
apt install linux-source-4.18.0
cd /usr/src/linux-source-4.18.0
```

# 四、进入内核源码并编译

```
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.17.11.tar.xz
wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.4.50.tar.xz
```

```
cd /linux-4.19.37-rt/linux-4.19.37-rt/

#清理环境
#make mrproper
#make clean

cp /boot/config-5.4.0-66-generic ./.config
make menuconfig
make -j8
make modules_install -j8
make install -j8

#cp /linux-4.19.37-rt/linux-4.19.37-rt/arch/arm64/boot/Image.gz /boot/vmlinuz-4.19.37-rt19.arm64
#cd /lib/modules
# 分析可加载模块的依赖性, 更新模块依赖
# depmod -a
#initramfs-4.19.37-rt19.img
#cd /boot
#dracut --kver 4.19.37-rt19
#lsinitrd initramfs-4.19.37-rt19.img | grep nvme
#dracut --kver 4.19.37-rt19 --add-drivers "nvme nvme_core"
#cp ./kernel_resource/config-4.19.37-rt19.arm64 /boot/config-4.19.37-rt19
#update-initramfs -c -k 5.4.50
#update-grub
```

```
vim /boot/efi/EFI/centos/grub.cfg

menuentry 'DeltaLinux LFS 3.19.37 (aarch64)' --class centos --class gnu-linux --class gnu --class os --unrestricted  {
        load_video
        set gfxpayload=keep
        insmod gzio
        insmod part_gpt
        insmod xfs
        set root='hd0,gpt2'
        if [ x$feature_platform_search_hint = xy ]; then
          search --no-floppy --fs-uuid --set=root --hint-ieee1275='ieee1275//disk@0,gpt2' --hint-bios=hd0,gpt2 --hint-efi=hd0,gpt2 --hint-baremetal=ahci0,gpt2  ccd31679-aeab-4dcf-a6bf-f2128b8b37d6
        else
          search --no-floppy --fs-uuid --set=root ccd31679-aeab-4dcf-a6bf-f2128b8b37d6
        fi
        linux /vmlinuz-4.19.37 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap LANG=en_US.UTF-8
        initrd /initramfs-4.19.37.img
}
```

```
update-ca-trust force-enable
update-ca-trust extract
openssl s_client -showcerts -connect github.com:443 </dev/null 2>/dev/null|openssl x509 -outform PEM >mycertfile.pem
cp mycertfile.pem /etc/pki/ca-trust/source/anchors/
update-ca-trust extract
```

```
objdump -tT libatlasutil.so
```

```
make config 基于文本模式的交互配置
make menuconfig 基于文本模式的菜单模式
make oldconfig
make defconfig
make localmodconfig
make clean
make distclean
```

# 五、内核操作

```
lsmod

modprobe vxlan

insmod ./kernel/drivers/net/vxlan.ko

modinfo ./kernel/drivers/net/vxlan.ko
```

# 六、内核镜像操作

```
mv initramfs-4.19.37-rt19.img.arm64 initramfs-4.19.37-rt19.img.arm64.gz
gzip -d initramfs-4.19.37-rt19.img.arm64.gz
cpio -id < initramfs-4.19.37-rt19.img.arm64

xz -dc initrd.img |cpio -id
kubernetes
find . |cpio -c -o |xz -9 --format=lzma > initrd.img

mksquashfs /mnt /root/install.img –all-root -noF
unsquashfs squashfs.img
```

# 七、FAQ

> chromium

```
Kernel Features -> Page size -> 4KB
```

> atlas

```
VDEV
ARM64_VA_BITS_48 

> CONFIG_ARM64_PAGE_SHIFT=16
> CONFIG_ARM64_CONT_SHIFT=5
> CONFIG_ARCH_MMAP_RND_BITS_MIN=14
> CONFIG_ARCH_MMAP_RND_BITS_MAX=29
> CONFIG_ARCH_MMAP_RND_COMPAT_BITS_MIN=7
> CONFIG_ARCH_MMAP_RND_COMPAT_BITS_MAX=16

vim arch/arm64/Kconfig
  │ Symbol: ARM64_VA_BITS_48 [=y]                                                                                                                                                                   │  
  │ Type  : bool                                                                                                                                                                                    │  
  │ Prompt: 48-bit                                                                                                                                                                                  │  
  │   Location:                                                                                                                                                                                     │  
  │     -> Kernel Features                                                                                                                                                                          │  
  │ (1)   -> Virtual address space size (<choice> [=y])                                                                                                                                             │  
  │   Defined at arch/arm64/Kconfig:658                                                                                                                                                             │  
  │   Depends on: <choice>                      

```

