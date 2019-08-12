---
layout: post
title: "Install Minikube"
date: 2019-08-12 11:15:32
image: '/assets/img/'
description:  '安装 Minikube'
main-class:  'tools'
color: 
tags:
 - docker
 - kubernetes
categories: 
- tools
twitter_text:  'simple process of Minikube installation'
introduction: 'simple process of Minikube installation'
---


# 前言

**[Minikube][minikube]** 是一个用来本地安装 **[Kubernetes][kubernetes]** 的简化工具 

>Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop for users looking to try out Kubernetes or develop with it day-to-day

当前 **[Kubernetes][kubernetes]** 非常火热，几乎成了新一代运维体系中的基础架构，为了对其特性进行了解，准备使用 **[Minikube][minikube]** 来完成对 **[Kubernetes][kubernetes]** 的安装

这里对 **[Minikube][minikube]** 的安装，进行一个简单的讲解

参考 **[Install Minikube][install_minikube]** 


> **Tip:** 当前的版本为 **minikube v1.3.0** 和 **kubectl v1.15.0**

---

# 操作


## 环境

~~~
root@pc:~# hostnamectl 
   Static hostname: pc
         Icon name: computer-laptop
           Chassis: laptop
        Machine ID: da43bae367934cabac43b08864bb91f9
           Boot ID: 24226098c59944dcbb52948e15b0eac1
  Operating System: Ubuntu 18.10
            Kernel: Linux 4.18.0-25-generic
      Architecture: x86-64
root@pc:~# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp109s0f1: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc fq_codel state DOWN group default qlen 1000
    link/ether 80:fa:5b:3b:d5:f4 brd ff:ff:ff:ff:ff:ff
3: wlp110s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 84:ef:18:f3:92:47 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.4/24 brd 10.0.0.255 scope global dynamic noprefixroute wlp110s0
       valid_lft 78725sec preferred_lft 78725sec
    inet6 fe80::5d3f:d6be:1fd9:4073/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
4: vboxnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 0a:00:27:00:00:00 brd ff:ff:ff:ff:ff:ff
    inet 10.1.0.1/24 brd 10.1.0.255 scope global vboxnet0
       valid_lft forever preferred_lft forever
    inet6 fe80::800:27ff:fe00:0/64 scope link 
       valid_lft forever preferred_lft forever
5: vboxnet1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 0a:00:27:00:00:01 brd ff:ff:ff:ff:ff:ff
root@pc:~# 
~~~

## 开始前的准备 

检查系统是否支持虚拟化

通过下面的命令来检查，看有没有 **`vmx|svm`** 指令集

~~~
root@pc:~# grep -E --color 'vmx|svm' /proc/cpuinfo
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
root@pc:~# 
~~~

## 安装 kubectl

参考 **[Install kubectl on Linux][kubectl]** 安装 **kubectl**

~~~
root@pc:~# curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.15.0/bin/linux/amd64/kubectl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 40.9M  100 40.9M    0     0  6468k      0  0:00:06  0:00:06 --:--:-- 7294k
root@pc:~# echo $?
0
root@pc:~#
root@pc:~# file kubectl 
kubectl: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, Go BuildID=einGj9I6M3H4OQuy-bjh/ujyMV7voNCHFA2QfI9Gb/5sgqz446kM2Asn787lMc/L6NPiCbVSghyh3NJH4YU, stripped
root@pc:~# ll kubectl 
-rw-r--r-- 1 root root 42985504 Ago 12 21:15 kubectl
root@pc:~# 
~~~

赋予执行权限

~~~
root@pc:~# ll kubectl 
-rw-r--r-- 1 root root 42985504 Ago 12 21:15 kubectl
root@pc:~# chmod +x ./kubectl 
root@pc:~# ll kubectl 
-rwxr-xr-x 1 root root 42985504 Ago 12 21:15 kubectl*
root@pc:~# 
~~~

移动到执行路径中

~~~
root@pc:~# ll kubectl 
-rw-r--r-- 1 root root 42985504 Ago 12 21:15 kubectl
root@pc:~# chmod +x ./kubectl 
root@pc:~# ll kubectl 
-rwxr-xr-x 1 root root 42985504 Ago 12 21:15 kubectl*
root@pc:~# 
~~~

查看版本

~~~
root@pc:~# kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:40:16Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
The connection to the server localhost:8080 was refused - did you specify the right host or port?
root@pc:~# 
~~~

**kubectl** 已经安装成功


## 安装 virtualbox

安装 **[VirtualBox][virtualbox]**   

这个过程就是一个软件包的安装过程，我这里就不演示了

~~~
root@pc:~# virtualbox -h
Oracle VM VirtualBox Manager 5.2.24
(C) 2005-2019 Oracle Corporation
All rights reserved.

Usage:
  --startvm <vmname|UUID>    start a VM by specifying its UUID or name
  --separate                 start a separate VM process
  --normal                   keep normal (windowed) mode during startup
  --fullscreen               switch to fullscreen mode during startup
  --seamless                 switch to seamless mode during startup
  --scale                    switch to scale mode during startup
  --no-startvm-errormsgbox   do not show a message box for VM start errors
  --restore-current          restore the current snapshot before starting
  --no-aggressive-caching    delays caching media info in VM processes
  --fda <image|none>         Mount the specified floppy image
  --dvd <image|none>         Mount the specified DVD image
  --dbg                      enable the GUI debug menu
  --debug                    like --dbg and show debug windows at VM startup
  --debug-command-line       like --dbg and show command line window at VM startup
  --debug-statistics         like --dbg and show statistics window at VM startup
  --no-debug                 disable the GUI debug menu and debug windows
  --start-paused             start the VM in the paused state
  --start-running            start the VM running (for overriding --debug*)

Expert options:
  --disable-patm             disable code patching (ignored by AMD-V/VT-x)
  --disable-csam             disable code scanning (ignored by AMD-V/VT-x)
  --recompile-supervisor     recompiled execution of supervisor code (*)
  --recompile-user           recompiled execution of user code (*)
  --recompile-all            recompiled execution of all code, with disabled
                             code patching and scanning
  --execute-all-in-iem       For debugging the interpreted execution mode.
  --warp-pct <pct>           time warp factor, 100% (= 1.0) = normal speed
  (*) For AMD-V/VT-x setups the effect is --recompile-all.

The following environment (and extra data) variables are evaluated:
  VBOX_GUI_DBG_ENABLED (GUI/Dbg/Enabled)
                             enable the GUI debug menu if set
  VBOX_GUI_DBG_AUTO_SHOW (GUI/Dbg/AutoShow)
                             show debug windows at VM startup
  VBOX_GUI_NO_DEBUGGER       disable the GUI debug menu and debug windows

root@pc:~#
~~~



**Minikube** 也可以在当前系统中使用，这样就不需要 hypervisor，但是需要使用软件包的方式安装 Docker ，不支持 snap 的 Docker 安装方式

>**Note:**  Minikube also supports a **`--vm-driver=none`** option that runs the Kubernetes components on the host and not in a VM. Using this driver requires Docker and a Linux environment but not a hypervisor. It is recommended to use the apt installation of docker from (Docker, when using the none driver. The snap installation of docker does not work with minikube.

## 安装 Minikube

~~~
root@pc:~# curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 53.1M  100 53.1M    0     0  5020k      0  0:00:10  0:00:10 --:--:-- 5360k
root@pc:~# echo $?
0
root@pc:~# ll minikube 
-rw-r--r-- 1 root root 55754504 Ago 12 22:39 minikube
root@pc:~# file minikube 
minikube: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, Go BuildID=tMjE5GbTZG3pynscXYnA/Z7r5TXGWDW5Jjnr-1aGV/Dp_bEdM72HUm6v9ZMT6g/bQg32CXSAa5-_KWYXJr2, BuildID[sha1]=5aa93ffca621e29f89597967c0166f249b685f4b, not stripped
root@pc:~# du -sh minikube 
54M	minikube
root@pc:~# chmod +x minikube 
root@pc:~# install minikube  /usr/local/bin
root@pc:~# 
root@pc:~# ll minikube 
-rwxr-xr-x 1 root root 55754504 Ago 12 22:39 minikube*
root@pc:~# mi
migrate-pubring-from-classic-gpg  min12xxw
mimeopen                          minfo
mimetype                          minikube
root@pc:~# 
root@pc:~# minikube -h
Minikube is a CLI tool that provisions and manages single-node Kubernetes
clusters optimized for development workflows.

Basic Commands:
  start          Starts a local kubernetes cluster
  status         Gets the status of a local kubernetes cluster
  stop           Stops a running local kubernetes cluster
  delete         Deletes a local kubernetes cluster
  dashboard      Access the kubernetes dashboard running within the minikube
cluster

Images Commands:
  docker-env     Sets up docker env variables; similar to '$(docker-machine
env)'
  cache          Add or delete an image from the local cache.

Configuration and Management Commands:
  addons         Modify minikube's kubernetes addons
  config         Modify minikube config
  profile        Profile gets or sets the current minikube profile
  update-context Verify the IP address of the running cluster in kubeconfig.

Networking and Connectivity Commands:
  service        Gets the kubernetes URL(s) for the specified service in your
local cluster
  tunnel         tunnel makes services of type LoadBalancer accessible on
localhost

Advanced Commands:
  mount          Mounts the specified directory into minikube
  ssh            Log into or run a command on a machine with SSH; similar to
'docker-machine ssh'
  kubectl        Run kubectl

Troubleshooting Commands:
  ssh-key        Retrieve the ssh identity key path of the specified cluster
  ip             Retrieves the IP address of the running cluster
  logs           Gets the logs of the running instance, used for debugging
minikube, not user code
  update-check   Print current and latest version number
  version        Print the version of minikube

Other Commands:
  completion     Outputs minikube shell completion for the given shell (bash or
zsh)

Use "minikube <command> --help" for more information about a given command.
root@pc:~# minikube version
minikube version: v1.3.0
commit: 43969594266d77b555a207b0f3e9b3fa1dc92b1f
root@pc:~# 
~~~

到此 **[Minikube][minikube]** 已经安装完毕


---

# 总结

所有的安装过程几乎都是文件下载赋权和改路径

非常简单


* TOC
{:toc}

---

[minikube]:https://kubernetes.io/docs/setup/learning-environment/minikube/
[kubernetes]:https://kubernetes.io/
[install_minikube]:https://kubernetes.io/docs/tasks/tools/install-minikube/
[kubectl]:https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux
[virtualbox]:https://www.virtualbox.org/wiki/Downloads



