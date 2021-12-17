# uboot

# 一、环境变量配置

```
setenv console 'console=ttyAMA1,115200'
setenv earlycon 'earlycon=pl011,0x28001000'
setenv rootfs 'root=/dev/sda3'
setenv ramdisk_size 'ramdisk_size=0x2000000'
setenv swapcnt 'swapaccount=1'
setenv cgrp_en 'cgroup_enable=memory'
setenv mmap3 'memmap=103M$0x90000000'
setenv mmap4 'memmap=7M$0x96000000'
setenv mamp5 'memmap=1024M$0xc0000000'
setenv boot_fdt 'booti 80100000 83000000 85000000'
setenv load_kernel fatload scsi 0:1 0x80100000 Image
setenv load_fdt fatload scsi 0:1 0x85000000 ft2004-devboard-d4-dsk.dtb
setenv load_ramdisk fatload scsi 0:1 83000000 initramfs-4.19.115.img.arm64-uboot
setenv bootargs ${console} ${earlycon} ${rootfs} rw quiet ${ramdisk_size} ${swapcnt} ${cgrp_en} nohugeiomap ${mmap3} ${mmap4} ${mamp5}
setenv distro_bootcmd 'run load_kernel; run load_fdt; run load_ramdisk; run boot_fdt'
saveenv
boot
```

# 二、常用命令

```
scsi
nvme
usb

scsi scan
scsi part

fatls scsi 0:1
 22784512   Image
    81836   initramfs-4.19.115.img.arm64-uboot
     9190   ft2004-devboard-d4-dsk.dtb

3 file(s), 0 dir(s)
```

