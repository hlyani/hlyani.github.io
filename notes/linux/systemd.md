# systemd

# type含义

```
Type=oneshot

这一选项适用于只执行一项任务、随后立即退出的服务。可能需要同时设置 RemainAfterExit=yes 使得 systemd 在服务进程退出之后仍然认为服务处于激活状态。

Type=notify

 与 Type=simple 相同，但约定服务会在就绪后向 systemd 发送一个信号。这一通知的实现由 libsystemd-daemon.so 提供。

Type=dbus

若以此方式启动，当指定的 BusName 出现在DBus系统总线上时，systemd认为服务就绪。

Type=idle systemd

会等待所有任务处理完成后，才开始执行 idle 类型的单元。其他行为与 Type=simple 类似。

Type=forking systemd

认为当该服务进程fork，且父进程退出后服务启动成功。对于常规的守护进程（daemon），除非你确定此启动方式无法满足需求，使用此类型启动即可。使用此启动类型应同时指定 PIDFile=，以便 systemd 能够跟踪服务的主进程

Type=simple

（默认值） systemd认为该服务将立即启动。服务进程不会 fork 。如果该服务要启动其他服务，不要使用此类型启动，除非该服务是socket 激活型。
```

