# abd

[https://adbdownload.com/](https://adbdownload.com/)

## 1、连接、安装

设备进入adb模式

```
adb connet 192.168.1.27
adb install *.apk
```

## 2、查看

```
adb devices
```

## 3、服务管理

```
adb kill-server
adb start-server
```

## 4、卸载

```
adb shell pm list packages
adb shell pm uninstall --user 0 APP_NAME
```