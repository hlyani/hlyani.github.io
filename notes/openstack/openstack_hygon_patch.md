# OpenStack对hygon的兼容

### 一、为什么要对OpenStack进行配置

海光处理器采用了与AMD EPYC 类似的体系结构，为了便于 QEMU 虚拟化处理器提供更好的兼容模式，需要对 QEMU-KVM进行CPU Vendor ID进行替换，进而支持 QEMU 虚拟机虚拟化 ；

QEMU在使用KVM虚拟化的时候只支持使用主机的CPU Vendor ID,目前海光处理器还不在此KVM虚拟化版本支持列表中，如果不配置就直接使用 QEMU/KVM，那么虚拟出来的虚拟机也为海光的 CPU VendorID，这将会导致某些OS虚拟机无法正常启动（例如Windows）；

Libvirt调用QEMU创建虚拟机进程，在配置Libvirt XML文件的时候需要指定CPU Model和VendorID，把这些参数传递给QEMU，虚拟机才能正常启动。

Openstack NOVA组件调用Libvirt接口控制虚拟机的生命周期，创建虚拟机的时候生成XML文件，原版的Openstack不会传递CPU的VendorID，需要做一些修改，传递相关的CPU参数才能生成正确的XML文件。

### 二、配置方法

##### 1、给nova_compute打补丁，将补丁文件Hygon_OpenStack_Train.patch拷贝到/usr/lib/python2.7/site-packages/nova目录

```
cd /var/lib/kolla/venv/lib/python2.7/site-packages/nova
```

##### 2、应用补丁

##### [补丁地址](https://github.com/hlyani/openstack_hygon_patch) 

```
git clone https://github.com/hlyani/openstack_hygon_patch.git
```

```
patch -p1 < Hygon_OpenStack_Train.patch
```

##### 3、重新nova-compute服务

```
systemctl restart openstack-nova-compute
```

##### 4、先查看cpu型号

```
# lscpu

Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                32
On-line CPU(s) list:   0-31
Thread(s) per core:    2
Core(s) per socket:    16
Socket(s):             1
NUMA node(s):          4
Vendor ID:             HygonGenuine
CPU family:            24
Model:                 0
Model name:            Hygon C86 7151 16-core Processor
Stepping:              1
CPU MHz:               1200.000
CPU max MHz:           2000.0000
CPU min MHz:           1200.0000
BogoMIPS:              3999.77
Virtualization:        AMD-V
L1d cache:             32K
L1i cache:             64K
L2 cache:              512K
L3 cache:              4096K
NUMA node0 CPU(s):     0-3,16-19
NUMA node1 CPU(s):     4-7,20-23
NUMA node2 CPU(s):     8-11,24-27
NUMA node3 CPU(s):     12-15,28-31
Flags:                 fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush mmx fxsr sse sse2 ht syscall nx mmxext fxsr_opt pdpe1gb rdtscp lm constant_tsc art rep_good nopl nonstop_tsc extd_apicid amd_dcm aperfmperf eagerfpu pni monitor ssse3 fma cx16 sse4_1 sse4_2 movbe popcnt xsave avx f16c rdrand lahf_lm cmp_legacy svm extapic cr8_legacy abm sse4a misalignsse 3dnowprefetch osvw skinit wdt tce topoext perfctr_core perfctr_nb bpext perfctr_l2 hw_pstate retpoline_amd ssbd ibpb vmmcall fsgsbase bmi1 avx2 smep bmi2 rdseed adx smap clflushopt xsaveopt xsavec xgetbv1 clzero irperf xsaveerptr arat npt lbrv svm_lock nrip_save tsc_scale vmcb_clean flushbyasid decodeassists pausefilter pfthreshold avic v_vmsave_vmload vgif overflow_recov succor smca
```

##### 5、创建新的flavor

```
nova flavor-create 4-8192-40-hygon 1000 8192 40 4
# win10内核的一个bug，win10对EPYC的支持有问题，hw:cpu_name='EPYC'
nova flavor-key 4-8192-40-hygon set hw:cpu_name='phenom'
nova flavor-key 4-8192-40-hygon set hw:cpu_vendor='AuthenticAMD'
nova flavor-key 4-8192-40-hygon set hw:cpu_model_id='Hygon C86 7151 16-core Processor'
nova flavor-key 4-8192-40-hygon set hw:cpu_sockets=2
nova flavor-key 4-8192-40-hygon set hw:cpu_cores=2
```

##### 6、基于新创的flavor创建虚拟机即可解决win10蓝屏问题