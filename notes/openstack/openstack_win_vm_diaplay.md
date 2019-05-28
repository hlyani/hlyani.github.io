# openstack windows虚拟机分辨率问题

```
glance image-upload --property hw_video_model=vga …… 
glance image-update --property hw_video_model=vga [image_uuid]
```
```
Libvirt
architecture - name of guest hardware architecture eg i686, x86_64, ppc64
hw_cdrom_bus - name of the CDROM bus to use eg virtio, scsi, ide
hw_disk_bus - name of the hard disk bus to use eg virtio, scsi, ide
hw_floppy_bus - name of the floppy disk bus to use eg fd, scsi, ide
hw_qemu_guest_agent - boolean 'yes' or 'no' to enable QEMU guest agent
hw_rng - name of the RNG device type eg virtio (pending merge)
hw_scsi_model - name of the SCSI bus controller eg 'virtio-scsi', 'lsilogic', etc (pending merge)
hw_video_model - name of the video adapter model to use, eg cirrus, vga, xen, qxl
hw_video_ram - MB of video RAM to provide eg 64 (pending merge)
hw_vif_model - name of a NIC device model eg virtio, e1000, rtl8139
hw_watchdog_action - action to take when watchdog device fires eg reset, poweroff, pause, none (pending merge)
os_command_line - string of boot time command line arguments for the guest kernel
VMWare
```
https://zhuanlan.zhihu.com/p/26670418

https://wiki.openstack.org/wiki/VirtDriverImageProperties