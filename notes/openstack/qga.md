# qemu-guest-agent 相关

##### 1、配置nova
```
nova image-meta image-id set hw_qemu_guest_agent=yes 
```
##### 2、配置glance
```
glance image-create --name cirros \
--disk-format raw \
--container-format bare \
--file cirros-0.3.3-x86_64-disk.raw \
--public \
--property hw_qemu_guest_agent=yes \
--progress
```
##### 3、可以通过命令来读取虚拟机内部真实 ip 
```
virsh qemu-agent-command instance-xxx '{"execute":"guest-network-get-interfaces"}' 

virsh qemu-agent-command instance-00000083 '{"execute":"guest-info"}' |python -m json.tool

virsh qemu-agent-command instance-00000083 '{"execute":"guest-file-open", "arguments":{"path":"/proc/cpuinfo","mode":"r"}}'

virsh qemu-agent-command instance-00000083 '{"execute":"guest-file-read", "arguments":{"handle":1003,"count":10000}}'
```
##### 4、已有功能
```
目前qga最新版本为1.5.50，linux已经实现下面的所有功能，windows仅支持加*的那些功能：

Ø guest-sync-delimited*：宿主机发送一个int数字给qga，qga返回这个数字，并且在后续返回字符串响应中加入ascii码为0xff的字符，其作用是检查宿主机与qga通信的同步状态，主要用在宿主机上多客户端与qga通信的情况下客户端间切换过程的状态同步检查，比如有两个客户端A、B，qga发送给A的响应，由于A已经退出，目前B连接到qga的socket，所以这个响应可能被B收到，如果B连接到socket之后，立即发送该请求给qga，响应中加入了这个同步码就能区分是A的响应还是B的响应；在qga返回宿主机客户端发送的int数字之前，qga返回的所有响应都要忽略；

Ø guest-sync*：与上面相同，只是不在响应中加入0xff字符；

Ø guest-ping*：Ping the guest agent, a non-error return implies success；

Ø guest-get-time*：获取虚拟机时间（返回值为相对于1970-01-01 in UTC，Time in nanoseconds.）；

Ø guest-set-time*：设置虚拟机时间（输入为相对于1970-01-01 in UTC，Time in nanoseconds.）；

Ø guest-info*：返回qga支持的所有命令；

Ø guest-shutdown*：关闭虚拟机（支持halt、powerdown、reboot，默认动作为powerdown）；

Ø guest-file-open：打开虚拟机内的某个文件（返回文件句柄）；

Ø guest-file-close：关闭打开的虚拟机内的文件；

Ø guest-file-read：根据文件句柄读取虚拟机内的文件内容（返回base64格式的文件内容）；

Ø guest-file-write：根据文件句柄写入文件内容到虚拟机内的文件；

Ø guest-file-seek：Seek to a position in the file, as with fseek(), and return the current file position afterward. Also encapsulates ftell()'s functionality, just Set offset=0, whence=SEEK_CUR；

Ø guest-file-flush：Write file changes bufferred in userspace to disk/kernel buffers；

Ø guest-fsfreeze-status：Get guest fsfreeze state. error state indicates；

Ø guest-fsfreeze-freeze：Sync and freeze all freezable, local guest filesystems；

Ø guest-fsfreeze-thaw：Unfreeze all frozen guest filesystems；

Ø guest-fstrim：Discard (or "trim") blocks which are not in use by the filesystem；

Ø guest-suspend-disk*：Suspend guest to disk；

Ø guest-suspend-ram*：Suspend guest to ram；

Ø guest-suspend-hybrid：Save guest state to disk and suspend to ram（This command requires the pm-utils package to be installed in the guest.）；

Ø guest-network-get-interfaces：Get list of guest IP addresses, MAC addresses and netmasks；

Ø guest-get-vcpus：Retrieve the list of the guest's logical processors；

guest-set-vcpus：Attempt to reconfigure (currently: enable/disable) logical processors inside the guest。
```