---
layout: post
title: "Minikube Enable Addons"
date: 2019-08-13 07:56:11
image: '/assets/img/'
description: 'Addons 的管理'
main-class:  'tools'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
- tools
twitter_text:  'enable addons by minikube'
introduction: 'enable addons by minikube'
---


# 前言

**[Minikube][minikube]** 是一个用来本地安装 **[Kubernetes][kubernetes]** 的简化工具 

>Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop for users looking to try out Kubernetes or develop with it day-to-day

当前 **[Kubernetes][kubernetes]** 非常火热，几乎成了新一代运维体系中的基础架构

为了对其特性进行了解

这里对 **[Minikube][minikube]** 的基础使用，进行一个简单的演示

通过 **[Minikube][minikube]** 来管理 Addons， Addons 可以用来扩展集群功能 

>Minikube has a set of built-in addons that can be enabled, disabled and opened in the local Kubernetes environment

参考 **[Hello Minikube][hello_minikube]** 

> **Tip:** 当前使用版本为 **minikube v1.2.0** 和 **kubectl v1.15.0**

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
nothing@pc:~$ ip addr
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
       valid_lft 62136sec preferred_lft 62136sec
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
nothing@pc:~$ minikube version
minikube version: v1.2.0
nothing@pc:~$ kubectl version
Client Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:40:16Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"15", GitVersion:"v1.15.0", GitCommit:"e8462b5b5dc2584fdcd18e6bcfe9f1e4d970a529", GitTreeState:"clean", BuildDate:"2019-06-19T16:32:14Z", GoVersion:"go1.12.5", Compiler:"gc", Platform:"linux/amd64"}
nothing@pc:~$ 
~~~

## 确保 k8s 集处于运行状态

查看状态

~~~
nothing@pc:~$minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.101
nothing@pc:~$ 
~~~

确保 k8s 集处于运行状态


## 展示当前集群中的所有插件

~~~
nothing@pc:~$ minikube addons list
- addon-manager: enabled
- dashboard: enabled
- default-storageclass: enabled
- efk: disabled
- freshpod: disabled
- gvisor: disabled
- heapster: disabled
- ingress: disabled
- logviewer: disabled
- metrics-server: disabled
- nvidia-driver-installer: disabled
- nvidia-gpu-device-plugin: disabled
- registry: disabled
- registry-creds: disabled
- storage-provisioner: enabled
- storage-provisioner-gluster: disabled
nothing@pc:~$
~~~

## 启用插件

Heapster 是容器集群监控和性能分析工具，天然的支持Kubernetes和CoreOS

我们使用 Minikube 来启用 Heapster

~~~
nothing@pc:~$ minikube addons enable heapster
* heapster was successfully enabled
nothing@pc:~$ 
~~~

再进行查看

~~~
nothing@pc:~$ minikube addons list | grep heap
- heapster: enabled
nothing@pc:~$ 
~~~

## 查看系统中的 service 和 pod

~~~
nothing@pc:~$ kubectl get pod,svc -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
pod/coredns-6967fb4995-72mcb              1/1     Running   2          5h26m
pod/coredns-6967fb4995-jfrdl              1/1     Running   2          5h26m
pod/etcd-minikube                         1/1     Running   1          5h25m
pod/heapster-fl8j7                        1/1     Running   0          54s
pod/influxdb-grafana-cv5zt                2/2     Running   0          54s
pod/kube-addon-manager-minikube           1/1     Running   1          5h25m
pod/kube-apiserver-minikube               1/1     Running   1          5h26m
pod/kube-controller-manager-minikube      1/1     Running   1          5h26m
pod/kube-proxy-2n2dl                      1/1     Running   1          5h26m
pod/kube-scheduler-minikube               1/1     Running   1          5h26m
pod/kubernetes-dashboard-95564f4f-fd7xd   1/1     Running   3          5h26m
pod/storage-provisioner                   1/1     Running   2          5h26m

NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
service/heapster               ClusterIP   10.109.13.67   <none>        80/TCP                   54s
service/kube-dns               ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP,9153/TCP   5h27m
service/kubernetes-dashboard   ClusterIP   10.111.67.35   <none>        80/TCP                   5h26m
service/monitoring-grafana     NodePort    10.96.65.96    <none>        80:30002/TCP             54s
service/monitoring-influxdb    ClusterIP   10.110.8.66    <none>        8083/TCP,8086/TCP        54s
nothing@pc:~$ 
~~~

## 禁用插件

~~~
nothing@pc:~$ minikube addons disable heapster
* heapster was successfully disabled
nothing@pc:~$ minikube addons list | grep heap
- heapster: disabled
nothing@pc:~$ 
~~~

再次查看系统中的 service 和 pod

~~~
nothing@pc:~$ kubectl get pod,svc -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
pod/coredns-6967fb4995-72mcb              1/1     Running   2          5h29m
pod/coredns-6967fb4995-jfrdl              1/1     Running   2          5h29m
pod/etcd-minikube                         1/1     Running   1          5h28m
pod/kube-addon-manager-minikube           1/1     Running   1          5h28m
pod/kube-apiserver-minikube               1/1     Running   1          5h28m
pod/kube-controller-manager-minikube      1/1     Running   1          5h28m
pod/kube-proxy-2n2dl                      1/1     Running   1          5h29m
pod/kube-scheduler-minikube               1/1     Running   1          5h28m
pod/kubernetes-dashboard-95564f4f-fd7xd   1/1     Running   3          5h29m
pod/storage-provisioner                   1/1     Running   2          5h29m

NAME                           TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                  AGE
service/kube-dns               ClusterIP   10.96.0.10     <none>        53/UDP,53/TCP,9153/TCP   5h29m
service/kubernetes-dashboard   ClusterIP   10.111.67.35   <none>        80/TCP                   5h29m
nothing@pc:~$ 
~~~

发现与 heapster 相关的服务和 pod 消失了

这就是简单的 addons 的管理

---

# 总结

使用 **minikube** 对 **k8s** 集群进行创建和销毁

使用 **kubectl** 对 **k8s** 集群对象或资源进行管理



* TOC
{:toc}

---

[minikube]:https://kubernetes.io/docs/setup/learning-environment/minikube/
[kubernetes]:https://kubernetes.io/
[hello_minikube]:https://kubernetes.io/docs/tutorials/hello-minikube/#enable-addons


