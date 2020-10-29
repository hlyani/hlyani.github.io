# Linux Desktop rdp 相关

# 一、SPICE、VNC、RDP三种协议对比

|              | SPICE                                                        | VNC                                                          | RDP                                                          |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| BIOS屏幕显示 | 能                                                           | 能                                                           | 不能                                                         |
| 全彩支持     | 能                                                           | 能                                                           | 能                                                           |
| 更改分辨率   | 能                                                           | 能                                                           | 能                                                           |
| 多显示器     | 多显示器支持（高达4画面）                                    | 只能一个屏幕                                                 | 多显示器支持                                                 |
| 图像传输     | 图像和图形传输                                               | 图像传输                                                     | 图像和图形传输                                               |
| 视频播放支持 | GPU加速支持                                                  | 不能                                                         | GPU加速支持                                                  |
| 音频传输     | 双向语音可以控制                                             | 不能                                                         | 双向语音可以控制                                             |
| 鼠标控制     | 客户端服务器都可以控制                                       | 服务器端控制                                                 | 服务器端控制                                                 |
| USB传输      | USB可以通过网络传输                                          | 不能                                                         | USB可以通过网络传输                                          |
| 适用系统     | Linux                                                        | Windows、Linux                                               | Windows、Linux                                               |
| 网络流量     | 较大、正常使用10-20M                                         | 较小、常用100K左右                                           | 较小、正常使用100-200K左右                                   |
| 适用场景     | 由于在色彩、音频和USB方面，适用于虚拟桌面，主要用于虚拟机的虚拟桌面应用。 | 主要用于Linux的服务器管理，由于无声音和USB传输，不满足于虚拟桌面的使用。 | 由于在色彩、音频、USB及本地磁盘映射方面较好，非常适用于虚拟桌面 |

# 二、安装 xrdp

##### 1、安装 xrdp

> CentOS

```
yum install -y epel-release
```

```
yum install -y xrdp
```

> Ubuntu

```
apt install -y xrdp
```

##### 2、启动 xrdp

```
systemctl start xrdp
systemctl enable xrdp
```

##### 3、查看 rdp 协议端口 3389

```
netstat -antup | grep xrdp
```

##### 4、其他，关闭防火墙或允许 3389

```
firewall-cmd --permanent --zone=public --add-port=3389/tcp
firewall-cmd –reload

# seliunux 设置
chcon --type=bin_t /usr/sbin/xrdp
chcon --type=bin_t /usr/sbin/xrdp-sesman
```

# 三、安装桌面

### 1、安装 xfce Desktop

> XFCE是最轻量级的桌面环境之一。它速度快，系统资源少，但在视觉上仍然很吸引人。此外，它有一个非常活跃的社区，因此有许多定制选项可用。

##### 1) 安装

```
yum install -y epel-release
yum groupinstall -y "Xfce"            
reboot
```

```
echo "xfce4-session" > ~/.Xclients
chmod a+x ~/.Xclients
```

##### 2）卸载

```
yum groupremove -y "Xfce"
yum remove -y libxfce4*
```

### 2、安装MATE Desktop

##### 1) 安装

```
yum install -y epel-release
yum groupinstall -y "MATE Desktop"
reboot
```

```
echo "mate-session" > ~/.Xclients
chmod a+x ~/.Xclients
```

##### 2）卸载

```
yum groupremove -y "MATE Desktop"
yum autoremove -y
```

### 3、安装 GNOME Desktop

##### 1) 安装

```
yum groupinstall "GNOME DESKTOP" -y
```

##### 2）启动

```
systemctl get-default
```

> 如果输出：multi-user.target，说明GUI没有被加载，需要将graphical.target设置为默认的target

```
systemctl set-default graphical.target
```

> 输出：
>
> Removed symlink /etc/systemd/system/default.target.
>
> Created symlink from /etc/systemd/system/default.target to /usr/lib/systemd/system/graphical.target.

```
systemctl isolate graphical.target
```

### 4、卸载

```
yum groupremove -y "GNOME Desktop"
yum autoremove -y
```

# 四、RDP 连接工具

##### 1、mRemoteNG

[mRemoteNG](https://github.com/mRemoteNG/mRemoteNG)

##### 2、multidesk

[multidesk](https://www.syvik.com/multidesk/index_chs.htm)

##### 3、remmia

> Debian/Ubuntu

```
apt-get install remmina remmina-plugin-*
```

> CentOS/RHEL

```
yum install remmina remmina-plugins-*
```

> Fedora 22

```
dnf copr enable hubbitus/remmina-next
dnf upgrade --refresh 'remmina*' 'freerdp*'
```

# 五、远程连接工具合集

| 工具名称                 | 支持平台 | 官网                                                     | 特点                                                     |
| ---------------------------- | ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| teamviewer               | windows  | [https://www.teamviewer.com/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cudGVhbXZpZXdlci5jb20v) | 远程桌面工具，私有远程tv协议                             |
| anydesk                  | windows  | [https://anydesk.com/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9hbnlkZXNrLmNvbS8=) | 类似teamviewer                                           |
| Radmin                   | windows  | [http://www.radmin.cn/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL3d3dy5yYWRtaW4uY24v) | 远程桌面工具                                             |
| xt800                    | windows  | [http://www.xt800.cn](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL3d3dy54dDgwMC5jbg==)/ | 国内首家支持多平台、多终端的远程运维和支持平台           |
| GoToMyPC                 | windows  | [https://gotomypc.en.softonic.com](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9nb3RvbXlwYy5lbi5zb2Z0b25pYy5jb20=) | GoToMyPC是美国Citrix公司推出的远程控制软件，类似于国内网络人远程控制软件。 |
| 网络人netman123          | windows  | [http://netman123.cn](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL25ldG1hbjEyMy5jbg==) |                                                              |
| Ammyy Admin              | windows  | [http://www.ammyy.com/cn/index.html](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL3d3dy5hbW15eS5jb20vY24vaW5kZXguaHRtbA==) |                                                              |
| FastX                    | windows  | [https://www.starnet.com/fastx](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cuc3Rhcm5ldC5jb20vZmFzdHg=) |                                                              |
| 3389远程服务器批量管理器 | windows  | [http://www.46603.cn/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cuNTJwb2ppZS5jbi90aHJlYWQtNDgwOTYxLTEtMS5odG1s) [https://www.52pojie.cn/thread-480961-1-1.html](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cuNTJwb2ppZS5jbi90aHJlYWQtNDgwOTYxLTEtMS5odG1s) | 批量连接windows桌面                                      |
| multidesk                | windows  | [http://www.syvik.com/multidesk/index_chs.htm](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL3d3dy5zeXZpay5jb20vbXVsdGlkZXNrL2luZGV4X2Nocy5odG0=) | 批量连接windows桌面                                      |
| Remmina                  | linux    | [http://www.remmina.org/wp/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL3d3dy5yZW1taW5hLm9yZy93cC8=) | Linux平台全协议RDP/VNC/SSH等                             |
| PAC Manager              | linux    | [https://sourceforge.net/projects/pacmanager/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9zb3VyY2Vmb3JnZS5uZXQvcHJvamVjdHMvcGFjbWFuYWdlci8=) | Linux平台全协议RDP/VNC/SSH等                             |
| mRemoteNG                | windows  | [https://github.com/mRemoteNG/mRemoteNG](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9naXRodWIuY29tL21SZW1vdGVORy9tUmVtb3RlTkc=) | win平台全协议RDP/VNC/SSH等                               |
| MobaXterm                | windows  | [https://mobaxterm.mobatek.net/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9tb2JheHRlcm0ubW9iYXRlay5uZXQv) | win平台全协议RDP/VNC/SSH等                               |
| FinalShell               | windows  | [http://www.hostbuf.com/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL3d3dy5ob3N0YnVmLmNvbS8=) | win平台全协议RDP/VNC/SSH等                               |
| Remote Desktop Manager   | windows  | [https://remotedesktopmanager.com/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9yZW1vdGVkZXNrdG9wbWFuYWdlci5jb20v) | win平台全协议RDP/VNC/SSH等                               |
| Terminals                | windows  | [https://github.com/terminals-Origin/Terminals](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9naXRodWIuY29tL3Rlcm1pbmFscy1PcmlnaW4vVGVybWluYWxz) | win平台全协议RDP/VNC/SSH等                               |
| Royal TSX                | windows  | [https://www.royalapplications.com/ts/win/features](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cucm95YWxhcHBsaWNhdGlvbnMuY29tL3RzL3dpbi9mZWF0dXJlcw==) | win平台全协议RDP/VNC/SSH等                               |
| putty                    | windows  | [https://www.putty.org/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cucHV0dHkub3JnLw==) | 简单的ssh工具                                            |
| KiTTY                    | windows  | [http://www.9bis.net/kitty/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL3d3dy45YmlzLm5ldC9raXR0eS8=) | fork自putty的ssh工具                                     |
| Cmder                    | windows  | [https://bliker.github.io/cmder/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9ibGlrZXIuZ2l0aHViLmlvL2NtZGVyLw==) | win下的命令行扩展，配合kitty使用                         |
| SecureCRT                | windows  | [https://www.vandyke.com/products/securecrt/index.html](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cudmFuZHlrZS5jb20vcHJvZHVjdHMvc2VjdXJlY3J0L2luZGV4Lmh0bWw=) | ssh的终端仿真工具                                        |
| VNC Connect              | 多平台   | [https://www.realvnc.com/en/connect/download/vnc/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cucmVhbHZuYy5jb20vZW4vY29ubmVjdC9kb3dubG9hZC92bmMv) | 连接VNC server的客户端                                   |
| rdesktop                 | linux    | [http://www.rdesktop.org/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL3d3dy5yZGVza3RvcC5vcmcv) | linux下远程连接windows的工具                             |
| ConnectBot               | 安卓     | [https://connectbot.org](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9jb25uZWN0Ym90Lm9yZw==) |                                                              |
| Microsoft Remote Desktop | 安卓/ios |                                                              | 微软官方提供的Windows连接工具                            |
| Bitvise SSH Client       | windows  | [https://www.bitvise.com/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cuYml0dmlzZS5jb20v) |                                                              |
| Xshell                   | windows  | [https://www.netsarang.com/products/xsh_overview.html](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cubmV0c2FyYW5nLmNvbS9wcm9kdWN0cy94c2hfb3ZlcnZpZXcuaHRtbA==) | widows下的ssh连接工具                                    |
| Chrome Secure Shell      | 浏览器   | [https://chrome.google.com/webstore/detail/secure-shell/pnhechapfaindjhompbnflcldabbghjo](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9jaHJvbWUuZ29vZ2xlLmNvbS93ZWJzdG9yZS9kZXRhaWwvc2VjdXJlLXNoZWxsL3BuaGVjaGFwZmFpbmRqaG9tcGJuZmxjbGRhYmJnaGpv) | 基于浏览器(Chrome)的ssh客户端                            |
| FIreSSH                  | 浏览器   | [http://firessh.net/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL2ZpcmVzc2gubmV0Lw==) | 基于浏览器(Firefox)的ssh客户端                           |
| Butterfly                | 浏览器   | [https://github.com/paradoxxxzero/butterfly](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly9naXRodWIuY29tL3BhcmFkb3h4eHplcm8vYnV0dGVyZmx5) | 浏览器中运行的xterm.js兼容终端                           |
| xterm.js                 | 浏览器   | [https://xtermjs.org/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly94dGVybWpzLm9yZy8=) | 基于浏览器的ssh客户端                                    |
| SSH Secure Shell Client  | windows  | [https://www.ssh.com/ssh/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cuc3NoLmNvbS9zc2gv) |                                                              |
| NoMachine                | 多平台   | [https://www.nomachine.com/](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cubm9tYWNoaW5lLmNvbS8=) | 私有远程NX协议、SSH协议                                  |
| xpra                     | 多平台   | [http://xpra.org](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cDovL3hwcmEub3Jn) | 私有远程xpra协议、SSH协议                                |
| winscp                   | windows  | [https://winscp.net/eng/docs/lang:chs](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93aW5zY3AubmV0L2VuZy9kb2NzL2xhbmc6Y2hz) | windows下向Linux的传输文件工具                           |
| Xftp                     | windows  | [https://www.netsarang.com/products/xfp_overview.html](https://www.yangxingzhen.com/wp-content/themes/begin/go.php?url=aHR0cHM6Ly93d3cubmV0c2FyYW5nLmNvbS9wcm9kdWN0cy94ZnBfb3ZlcnZpZXcuaHRtbA==) | windows下向Linux的传输文件工具                           |