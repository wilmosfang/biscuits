---
layout: post
title: "Setup A K8S deployment"
date: 2020-03-07 16:23:11
image: '/assets/img/'
description: '启动一个 k8s 应用'
main-class: kubernetes
color: 
tags: 
 - tools
 - kubernetes
categories: 
 - kubernetes
twitter_text: 'Setup A K8S deployment'
introduction: 'Setup A K8S deployment'
---

# 前言

这里通过一次 workshop 来演示如何在 K8s 中启动一个应用

>**Tip:** 当前的版本为 

~~~
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:58:59Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.0", GitCommit:"9e991415386e4cf155a24b1da15becaa390438d8", GitTreeState:"clean", BuildDate:"2020-03-25T14:50:46Z", GoVersion:"go1.13.8", Compiler:"gc", Platform:"linux/amd64"}
~~~

---

# 操作

## 系统环境

~~~
➜  k8s_C7X64 git:(master) sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.3
BuildVersion:	19D76
➜  k8s_C7X64 git:(master) vagrant version
Installed Version: 2.2.5
Latest Version: 2.2.7

To upgrade to the latest version, visit the downloads page and
download and install the latest version of Vagrant from the URL
below:

  https://www.vagrantup.com/downloads.html

If you're curious what changed in the latest release, view the
CHANGELOG below:

  https://github.com/hashicorp/vagrant/blob/v2.2.7/CHANGELOG.md
➜  k8s_C7X64 git:(master)
~~~


当前的节点状态

~~~
[vagrant@m1 ~]$ kubectl get nodes
NAME   STATUS   ROLES    AGE     VERSION
m1     Ready    master   20m     v1.18.0
n1     Ready    <none>   3m59s   v1.18.0
n2     Ready    <none>   41s     v1.18.0
[vagrant@m1 ~]$
~~~

## 运行一个 Deployment

~~~
[vagrant@m1 ~]$ kubectl run nginx-deployment --image=nginx:1.7.9
pod/nginx-deployment created
[vagrant@m1 ~]$ kubectl get pods
NAME               READY   STATUS              RESTARTS   AGE
nginx-deployment   0/1     ContainerCreating   0          7s
[vagrant@m1 ~]$ kubectl get pods
NAME               READY   STATUS              RESTARTS   AGE
nginx-deployment   0/1     ContainerCreating   0          13s
[vagrant@m1 ~]$ kubectl get pods
NAME               READY   STATUS              RESTARTS   AGE
nginx-deployment   0/1     ContainerCreating   0          15s
[vagrant@m1 ~]$ kubectl get pods
NAME               READY   STATUS    RESTARTS   AGE
nginx-deployment   1/1     Running   0          16s
[vagrant@m1 ~]$
~~~

发现所有的 pod 都处于 Running 状态

到这里 K8s 3节点的集群就已经安装完毕，并且处于正常运行状态

---

# 总结

这里简单演示了如何使用 kubeadm 配置一个 k8s 集群

后面我们接着演示如何运行应用

* TOC
{:toc}


---


