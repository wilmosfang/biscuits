---
layout: post
title: "Creating a Cluster"
date: 2019-08-13 12:20:13
image: '/assets/img/'
description: '创建一个 K8s 集群'
main-class:  'tools'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
- tools
twitter_text:  'create a kubernetes cluster'
introduction: 'create a kubernetes cluster'
---


# 前言


这节课通过 **[minikube][minikube]** 来构建一个 **[k8s][kubernetes]** 集群

>The goal of this interactive scenario is to deploy a local development Kubernetes cluster using minikube


参考 **[Create a Kubernetes cluster][create_cluster]** 


> **Tip:** 当前的版本为 **minikube v1.2.0** 和 **kubectl v1.15.0**

---

# 操作


## 环境

~~~
nothing@pc:~$ hostnamectl 
   Static hostname: pc
         Icon name: computer-laptop
           Chassis: laptop
        Machine ID: da43bae367934cabac43b08864bb91f9
           Boot ID: e0a360e3552246c5b952cb2f245c43a9
  Operating System: Ubuntu 18.10
            Kernel: Linux 4.18.0-25-generic
      Architecture: x86-64
nothing@pc:~$ ip a 
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
       valid_lft 77493sec preferred_lft 77493sec
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
6: vboxnet2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 0a:00:27:00:00:02 brd ff:ff:ff:ff:ff:ff
    inet 192.168.99.1/24 brd 192.168.99.255 scope global vboxnet2
       valid_lft forever preferred_lft forever
    inet6 fe80::800:27ff:fe00:2/64 scope link 
       valid_lft forever preferred_lft forever
nothing@pc:~$ 
~~~

## 开始前的准备 

检查系统是否支持虚拟化

通过下面的命令来检查，看有没有 **`vmx|svm`** 指令集

~~~
nothing@pc:~$ grep -E --color 'vmx|svm' /proc/cpuinfo
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
flags		: fpu vme de pse tsc msr pae mce cx8 apic sep mtrr pge mca cmov pat pse36 clflush dts acpi mmx fxsr sse sse2 ss ht tm pbe syscall nx pdpe1gb rdtscp lm constant_tsc art arch_perfmon pebs bts rep_good nopl xtopology nonstop_tsc cpuid aperfmperf tsc_known_freq pni pclmulqdq dtes64 monitor ds_cpl vmx est tm2 ssse3 sdbg fma cx16 xtpr pdcm pcid sse4_1 sse4_2 x2apic movbe popcnt tsc_deadline_timer aes xsave avx f16c rdrand lahf_lm abm 3dnowprefetch cpuid_fault epb invpcid_single pti ssbd ibrs ibpb stibp tpr_shadow vnmi flexpriority ept vpid fsgsbase tsc_adjust bmi1 hle avx2 smep bmi2 erms invpcid rtm mpx rdseed adx smap clflushopt intel_pt xsaveopt xsavec xgetbv1 xsaves dtherm ida arat pln pts hwp hwp_notify hwp_act_window hwp_epp md_clear flush_l1d
nothing@pc:~$ 
~~~

## 安装 virtualbox

安装 **[VirtualBox][virtualbox]**   

这个过程就是一个软件包的安装过程，我这里就不演示了

~~~
nothing@pc:~$ virtualbox -h
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

nothing@pc:~$ 
~~~



**Minikube** 也可以在当前系统中使用，这样就不需要 hypervisor，但是需要使用软件包的方式安装 Docker ，不支持 snap 的 Docker 安装方式

>**Note:**  Minikube also supports a **`--vm-driver=none`** option that runs the Kubernetes components on the host and not in a VM. Using this driver requires Docker and a Linux environment but not a hypervisor. It is recommended to use the apt installation of docker from (Docker, when using the none driver. The snap installation of docker does not work with minikube.

## 安装 Minikube

~~~
nothing@pc:~$ curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.2.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 39.8M  100 39.8M    0     0  6619k      0  0:00:06  0:00:06 --:--:-- 7247k
[sudo] nothing 的密码： 
nothing@pc:~$ 
nothing@pc:~$ minikube version
minikube version: v1.2.0
nothing@pc:~$
~~~

到此 **[Minikube][minikube]** 已经安装完毕


## 启动一个 k8s 集群

~~~
nothing@pc:~$ minikube start --registry-mirror=https://registry.docker-cn.com
* minikube v1.2.0 on linux (amd64)
* using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
* Downloading Minikube ISO ...
 129.33 MB / 129.33 MB [============================================] 100.00% 0s
* Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
* Configuring environment for Kubernetes v1.15.0 on Docker 18.09.6
* Downloading kubeadm v1.15.0
* Downloading kubelet v1.15.0
* Pulling images ...
* Launching Kubernetes ... 
* Verifying: apiserver proxy etcd scheduler controller dns
* Done! kubectl is now configured to use "minikube"
nothing@pc:~$ 
~~~

![k8s](/assets/img/k8s/k8s01.png)

查看状态

~~~
nothing@pc:~$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.102
nothing@pc:~$ 
~~~

Minikube 已经创建了一个虚拟机，并且一个 k8s 集群已经运行其中


## 查看 kubectl 版本

~~~
nothing@pc:~$ kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:40:16Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:32:14Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
nothing@pc:~$ 
~~~

**kubectl** 是与 k8s 集群交互的 CLI 工具

这个命令里提供了 client 和 server 端的版本

## 查看集群信息

~~~
nothing@pc:~$ kubectl cluster-info
Kubernetes master is running at https://192.168.99.102:8443
KubeDNS is running at https://192.168.99.102:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
nothing@pc:~$
~~~

可以使用  **`kubectl cluster-info dump`** 查看更详细的集群信息

现在已经在集群中运行了一个 master 和 一个 dashboard

Kubernetes dashboard 可以通过 web UI 来访问

但是这里我们只使用 CLI 的方式来完成所有管理 


## 查看 node

~~~
nothing@pc:~$ kubectl get nodes
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   8m23s   v1.15.0
nothing@pc:~$
~~~

这个命令展示了我们系统中可用的节点，这些节点可以用来运行我们的应用

现在我们只有一个节点，它的状态是 Ready 的，代表了它已经可以开始承载业务了


---

# 总结

主要是 kubectl 命令的使用

~~~
minikube version
minikube start --registry-mirror=https://registry.docker-cn.com
minikube status
kubectl version
kubectl cluster-info
kubectl cluster-info dump
kubectl get nodes
~~~

* TOC
{:toc}

---


[create_cluster]:https://kubernetes.io/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/

[minikube]:https://kubernetes.io/docs/setup/learning-environment/minikube/
[kubernetes]:https://kubernetes.io/
[install_minikube]:https://yq.aliyun.com/articles/508460
[kubectl]:https://kubernetes.io/docs/tasks/tools/install-kubectl/#install-kubectl-on-linux
[virtualbox]:https://www.virtualbox.org/wiki/Downloads


