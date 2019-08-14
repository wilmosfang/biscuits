---
layout: post
title: "Scaling The App"
date: 2019-08-14 04:28:45
image: '/assets/img/'
description: '扩展应用'
main-class:  'tools'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
- tools
twitter_text:  'scaling the app'
introduction: 'scaling the app'
---




# 前言


这篇讲如何使用 kubectl 来申缩应用

>The goal of this interactive scenario is to scale a deployment with kubectl scale and to see the load balancing in action

参考 **[Scaling Your App][scale_interactive]** 


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
       valid_lft 73821sec preferred_lft 73821sec
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

## 查看当前 deployment 状态

~~~
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/1     1            1           14h
nothing@pc:~$ 
~~~

这里表明，有一个 Pod

**READY**  代表当前运行中的和期望复本数的比例

**CURRENT** 代表正在运行中的复本数量

**DESIRED** 代表配置中的复本数量

**UP-TO-DATE** 代表达到设定状态的复本数

**AVAILABLE** 代表有多少正在提供服务的复本数


## 扩展复本数

~~~
nothing@pc:~$ kubectl scale deployments/kubernetes-bootcamp --replicas=4
deployment.extensions/kubernetes-bootcamp scaled
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   1/4     4            1           14h
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           14h
nothing@pc:~$
~~~

我们使用 scale 命令扩展复本数到 4 个

一旦新的变更应用，我们就拥有了4 个可用实例

## 查看系统中的 pods

~~~
nothing@pc:~$ kubectl get pods 
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fff89cb6-42cpz   1/1     Running   1          14h
kubernetes-bootcamp-69fff89cb6-hxwdt   1/1     Running   0          3m35s
kubernetes-bootcamp-69fff89cb6-lbl2w   1/1     Running   0          3m35s
kubernetes-bootcamp-69fff89cb6-z5dtt   1/1     Running   0          3m35s
nothing@pc:~$ kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
kubernetes-bootcamp-69fff89cb6-42cpz   1/1     Running   1          14h     172.17.0.5   minikube   <none>           <none>
kubernetes-bootcamp-69fff89cb6-hxwdt   1/1     Running   0          3m36s   172.17.0.6   minikube   <none>           <none>
kubernetes-bootcamp-69fff89cb6-lbl2w   1/1     Running   0          3m36s   172.17.0.8   minikube   <none>           <none>
kubernetes-bootcamp-69fff89cb6-z5dtt   1/1     Running   0          3m36s   172.17.0.7   minikube   <none>           <none>
nothing@pc:~$ 
~~~

现在拥有了 4 个不同IP 的 Pods


## 查看 deployments 的事件

~~~
nothing@pc:~$ kubectl describe deployments
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Tue, 13 Aug 2019 22:29:02 +0800
Labels:                 run=kubernetes-bootcamp
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=kubernetes-bootcamp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=kubernetes-bootcamp
  Containers:
   kubernetes-bootcamp:
    Image:        daocloud.io/library/nginx
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kubernetes-bootcamp-69fff89cb6 (4/4 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  5m39s  deployment-controller  Scaled up replica set kubernetes-bootcamp-69fff89cb6 to 4
nothing@pc:~$ 
~~~

事件栏中多了一条关于扩展复本数的事件

## 负载均衡


创建一个 service 

~~~
nothing@pc:~$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 80
service/kubernetes-bootcamp exposed
nothing@pc:~$ kubectl describe services/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.104.250.0
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30910/TCP
Endpoints:                172.17.0.5:80,172.17.0.6:80,172.17.0.7:80 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
nothing@pc:~$ 
~~~

将端口存到环境变量里

~~~
nothing@pc:~$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{\{(index .spec.ports 0).nodePort}\}')
nothing@pc:~$ echo NODE_PORT=$NODE_PORT
NODE_PORT=30910
nothing@pc:~$ 
~~~

## 进行访问

~~~
nothing@pc:~$ curl $(minikube ip):$NODE_PORT
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
nothing@pc:~$ 
~~~

这里有每一次访问都负载在不同的 pod 中

## 缩容

~~~
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fff89cb6-42cpz   1/1     Running   1          14h
kubernetes-bootcamp-69fff89cb6-hxwdt   1/1     Running   0          16m
kubernetes-bootcamp-69fff89cb6-lbl2w   1/1     Running   0          16m
kubernetes-bootcamp-69fff89cb6-z5dtt   1/1     Running   0          16m
nothing@pc:~$ kubectl scale deployments/kubernetes-bootcamp --replicas=2
deployment.extensions/kubernetes-bootcamp scaled
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-69fff89cb6-42cpz   1/1     Running       1          14h
kubernetes-bootcamp-69fff89cb6-hxwdt   1/1     Running       0          17m
kubernetes-bootcamp-69fff89cb6-lbl2w   0/1     Terminating   0          17m
kubernetes-bootcamp-69fff89cb6-z5dtt   0/1     Terminating   0          17m
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fff89cb6-42cpz   1/1     Running   1          14h
kubernetes-bootcamp-69fff89cb6-hxwdt   1/1     Running   0          17m
nothing@pc:~$ 
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   2/2     2            2           14h
nothing@pc:~$ 
nothing@pc:~$ kubectl get pods -o wide
NAME                                   READY   STATUS    RESTARTS   AGE   IP           NODE       NOMINATED NODE   READINESS GATES
kubernetes-bootcamp-69fff89cb6-42cpz   1/1     Running   1          14h   172.17.0.5   minikube   <none>           <none>
kubernetes-bootcamp-69fff89cb6-hxwdt   1/1     Running   0          20m   172.17.0.6   minikube   <none>           <none>
nothing@pc:~$
~~~

同样使用 **`scale`** 来进行缩容

---

# 总结

主要是 kubectl 命令的使用

~~~
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 80
kubectl describe services/kubernetes-bootcamp
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{\{(index .spec.ports 0).nodePort}\}')
echo NODE_PORT=$NODE_PORT
curl $(minikube ip):$NODE_PORT
kubectl get pods
kubectl scale deployments/kubernetes-bootcamp --replicas=2
kubectl get deployments
kubectl get pods -o wide
~~~

* TOC
{:toc}

---


[scale_interactive]:https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-interactive/
[kubernetes]:https://kubernetes.io/


