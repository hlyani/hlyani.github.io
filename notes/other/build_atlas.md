# Atlas 驱动构建

# 一、准备内核SDK

略

# 二、安装编译环境

```
DEBIAN_FRONTEND=noninteractive apt install -y gcc dracut git make libncurses-dev libelf-dev bison flex libssl-dev bc u-boot-tools vim
```

# 三、编译

[atlas](https://github.com/hlyani/atlas)

## 1、下载解压 makeself

```
mkdir rebuild
cd rebuild
wget https://github.com/megastep/makeself/archive/release-2.4.0.tar.gz
tar -zxvf release-2.4.0.tar.gz
```

## 2、解压driver驱动包

```
./A300-3000-3010-npu-driver_20.2.0_linux-aarch64.run --noexec --extract=./tmp
```

## 3、解压repack包

```
tar -zxvf A300-3000-npu-driver-repack-tools_20.2.0.tar.gz --strip-components 1
```

## 4、重构

```
bash build.sh ./makeself-release-2.4.0/makeself.sh tmp/ A300-3000-3010-npu-driver_20.2.0_linux_XXX-aarch64.run
```

