# 区分当前系统运行环境

> demidecode是一个可以将DMI table中的内容以人类可读格式导出的工具。DMI（也被称为SMBIOS）Table中保存的是该表包含系统硬件组成的描述，以及其他有用的信息，例如序列号和BIOS版本等。

# 一、物理机

```
dmidecode -s system-product-name

X10DRH
```

# 二、虚拟机

## 1、kvm

```
dmidecode -s system-product-name

KVM
```

## 2、OpenStack

```
dmidecode -s system-product-name

OpenStack Nova
```

### 3、VMware

```
dmidecode -s system-product-name

VMware Virtual Platform
```

## 4、VirtualBox

```
dmidecode -s system-product-name

VirtualBox
```

# 三、容器

## 1、docker

> docker 容器通常会在"/"目录下有个一个dockerenv文件。老版本可能是dockerinit文件。

```
ls /*docker*

/.dockerenv
```

> 或者直接通过查看cgroup信息中是否包含docker字段。

```
cat /proc/1/cgroup
```

## 2、K8S

> 查看环境变量

```
env | grep KUBE
```

>如果有根目录下有docker文件，且env有k8s环境变量说明容器运行时使用docker，否则是containerd。可通过cgroup进一步确认。

```
cat /proc/1/cgroup|grep docker

11:pids:/kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod10cb94c2_305c_4edd_99d0_09428515a8c7.slice/cri-containerd-272a744e650bf5f6e13a3f83ee21f9f8bb4d8a527d3656db057e1f21b10b0871.scope
```

