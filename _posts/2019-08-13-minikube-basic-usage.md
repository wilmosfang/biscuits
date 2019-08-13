---
layout: post
title: "Minikube Basic Usage"
date: 2019-08-13 02:36:54
image: '/assets/img/'
description: 'Minikube 的基础用法'
main-class:  'tools'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
- tools
twitter_text:  'Basic Usage of Minikube '
introduction: 'Basic Usage of Minikube'
---

# 前言

**[Minikube][minikube]** 是一个用来本地安装 **[Kubernetes][kubernetes]** 的简化工具 

>Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop for users looking to try out Kubernetes or develop with it day-to-day

当前 **[Kubernetes][kubernetes]** 非常火热，几乎成了新一代运维体系中的基础架构

为了对其特性进行了解

这里对 **[Minikube][minikube]** 的基础使用，进行一个简单的演示



参考 **[Installing Kubernetes with Minikube][usage_minikube]** 


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
       valid_lft 77558sec preferred_lft 77558sec
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

## 清理现有集群资源

~~~
nothing@pc:~$ minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.100
nothing@pc:~$ minikube  stop 
* Stopping "minikube" in virtualbox ...
* "minikube" stopped.
nothing@pc:~$ minikube delete
* Deleting "minikube" from virtualbox ...
* The "minikube" cluster has been deleted.
nothing@pc:~$
~~~


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
nothing@pc:~$ echo $?
0
nothing@pc:~$ 
~~~


![k8s](/assets/img/k8s/k8s01.png)

查看状态

~~~
nothing@pc:~$minikube status
host: Running
kubelet: Running
apiserver: Running
kubectl: Correctly Configured: pointing to minikube-vm at 192.168.99.101
nothing@pc:~$ 
~~~

已经创建了一个 k8s 集群

## 创建一个 Kubernetes Deployment

~~~
nothing@pc:~$ kubectl run hello-nginx --image=daocloud.io/library/nginx --port=80
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/hello-nginx created
nothing@pc:~$ 
~~~

## 发布这个 Deployment

~~~
nothing@pc:~$ kubectl expose deployment hello-nginx --type=NodePort
service/hello-nginx exposed
nothing@pc:~$
~~~

通过 **`--type=NodePort`** 来指定 Service 的类型

## 查看 pod 对象

~~~
nothing@pc:~$ kubectl get pod
NAME                           READY   STATUS    RESTARTS   AGE
hello-nginx-688bf9cccd-k7kft   1/1     Running   0          2m30s
nothing@pc:~$ 
~~~

从 **STATUS** 看已经处于 **Running** 

##  查看 service 的访问路径

~~~
nothing@pc:~$ minikube service hello-nginx --url
http://192.168.99.101:30147
nothing@pc:~$ 
~~~

也可以通过 kubectl 来查看

~~~
nothing@pc:~$ kubectl get service
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
hello-nginx   NodePort    10.99.203.171   <none>        80:30147/TCP   4m54s
kubernetes    ClusterIP   10.96.0.1       <none>        443/TCP        13m
nothing@pc:~$ 
~~~

## 在浏览器中进行访问

![k8s](/assets/img/k8s/k8s02.png)


## 删除 service

~~~
nothing@pc:~$ kubectl delete services hello-nginx
service "hello-nginx" deleted
nothing@pc:~$ 
~~~

## 删除 deployment

~~~
nothing@pc:~$ kubectl delete deployment hello-nginx
deployment.extensions "hello-nginx" deleted
nothing@pc:~$ 
~~~


## 停止 minikube 集群

~~~
nothing@pc:~$ minikube stop 
* Stopping "minikube" in virtualbox ...
* "minikube" stopped.
nothing@pc:~$ 
~~~

![k8s](/assets/img/k8s/k8s03.png)


## 启动 minikube 集群

~~~
nothing@pc:~$ minikube start 
* minikube v1.2.0 on linux (amd64)
* using image repository registry.cn-hangzhou.aliyuncs.com/google_containers
* Tip: Use 'minikube start -p <name>' to create a new cluster, or 'minikube delete' to delete this one.
* Restarting existing virtualbox VM for "minikube" ...
* Waiting for SSH access ...
* Configuring environment for Kubernetes v1.15.0 on Docker 18.09.6
* Relaunching Kubernetes v1.15.0 using kubeadm ... 
* Verifying: apiserver proxy etcd scheduler controller dns
* Done! kubectl is now configured to use "minikube"
nothing@pc:~$
~~~

![k8s](/assets/img/k8s/k8s01.png)

---

# 总结

使用 **minikube** 对 **k8s** 集群进行创建和销毁

使用 **kubectl** 对 **k8s** 集群对象或资源进行管理


~~~
* minikube version
* kubectl version
* minikube status
* minikube  stop 
* minikube delete
* minikube help 
* minikube start --registry-mirror=https://registry.docker-cn.com
* kubectl run hello-nginx --image=daocloud.io/library/nginx --port=80
* kubectl expose deployment hello-nginx --type=NodePort
* kubectl get pod
* minikube service hello-nginx --url
* kubectl get service
* kubectl get service hello-nginx
* kubectl delete services hello-nginx
* kubectl delete deployment hello-nginx
* kubectl get pods
* kubectl get deployment
* minikube start 
~~~


* TOC
{:toc}

---

[minikube]:https://kubernetes.io/docs/setup/learning-environment/minikube/
[kubernetes]:https://kubernetes.io/
[usage_minikube]:https://kubernetes.io/docs/setup/learning-environment/minikube/


