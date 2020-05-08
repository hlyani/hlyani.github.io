# 挂载USB到docker容器

### 一、基础准备

##### 1、部署 docker 环境，拉取或构建带vnc的docker镜像

[https://hub.docker.com/r/x11vnc/desktop](https://hub.docker.com/r/x11vnc/desktop)

```
git clone https://github.com/hlyani/docker-ubuntu-vnc-desktop.git
```

```
git clone https://github.com/hlyani/x11vnc-desktop.git
```

```
docker pull x11vnc/desktop
```

```
docker pull centminmod/docker-ubuntu-vnc-desktop
```

##### 3、查看主机 usb 设备

```
apt -y install usbutils
```

```
lsusb  -D /dev/bus/usb/001/001
lsusb -s 001:001
```

##### 4、查看视频设备

```
ls /dev/video*
```

##### 5、运行 docker

> 在运行docker时添加--privileged，这样可以放开 docker 的所有系统操作权限，但是这种操作带来的安全风险比较大。

```
docker run -itu0 --rm -p 6080:6080 --privileged=true --name test --device=/dev/video0 --device=/dev/video1 -v /dev/bus/usb:/dev/bus/usb x11vnc/desktop
```

```
docker run -itu0 --name test --privileged=true --net=host -v /dev/bus/usb:/dev/bus/usb ubuntu bash
```

```
docker run -itu0 --name test --privileged=true --net=host -v /dev/bus/usb:/dev/bus/usb ubuntu bash
```

```
docker run -it --rm --device=/dev/video0 -e DISPLAY=unix$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix whynot0/opencamrea:v1
```

```
docker run -it --rm --runtime=nvidia -v /dev/bus/usb:/dev/bus/usb -e NVIDIA_VISIBLE_DEVICES=0 -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=unix$DISPLAY cwaffles/openpose-python
```

```
docker run -t -i --device=/dev/ttyUSB0 ubuntu bash
```

### 二、挂载摄像头并显示视频

[https://blog.csdn.net/weixin_40922744/article/details/103245166](https://blog.csdn.net/weixin_40922744/article/details/103245166)

[https://yq.aliyun.com/articles/606756 ](https://yq.aliyun.com/articles/606756)

##### 1、安装摄像头第三方软件

```
apt install -y camorama
```

```
apt install -y cheese
```

##### 2、运行

```
camorama
```

```
cheese
```

```
apt-get install guvcview
guvcview -d /dev/video
```

