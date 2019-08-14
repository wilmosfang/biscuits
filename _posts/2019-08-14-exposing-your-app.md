---
layout: post
title: "Exposing the App"
date: 2019-08-14 01:13:55
image: '/assets/img/'
description: '发布应用'
main-class:  'tools'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
- tools
twitter_text:  'exposing the app'
introduction: 'exposing the app'
---



# 前言


这篇讲如何使用 kubectl 来发布应用到外网访问，如何使用 kubectl 命令来给对象打标签

>In this scenario you will learn how to expose Kubernetes applications outside the cluster using the kubectl expose command. You will also learn how to view and apply labels to objects with the kubectl label command

参考 **[Exposing Your App][expose_interactive]** 


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

## 查看 pods 运行状态


我们可以使用 get pods 的方式来查看集群中的 pods 状态

查看正在运行中的应用

~~~
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fff89cb6-42cpz   1/1     Running   1          12h
nothing@pc:~$ 
~~~

## 查看集群中的 services

~~~
nothing@pc:~$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   14h
nothing@pc:~$ 
nothing@pc:~$ kubectl describe services
Name:              kubernetes
Namespace:         default
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP:                10.96.0.1
Port:              https  443/TCP
TargetPort:        8443/TCP
Endpoints:         192.168.99.100:8443
Session Affinity:  None
Events:            <none>
nothing@pc:~$
~~~

系统中有一个 kubernetes 集群的 services，是由 minikube 创建的

## 创建一个 services

~~~
nothing@pc:~$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 80
service/kubernetes-bootcamp exposed
nothing@pc:~$ kubectl get services
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP        14h
kubernetes-bootcamp   NodePort    10.98.252.197   <none>        80:32070/TCP   8s
nothing@pc:~$
~~~

使用 expose 命令以 **NodePort** 的方式将服务发布出去


这样我们就创建了一个 service，通过  describe 来查看详细的信息


~~~
nothing@pc:~$ kubectl describe service kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.98.252.197
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  32070/TCP
Endpoints:                172.17.0.5:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
nothing@pc:~$
~~~

describe 可以看到更多维度的信息


## 进行访问

先将端口存下来

~~~
nothing@pc:~$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{\{(index .spec.ports 0).nodePort}\}')
nothing@pc:~$ echo NODE_PORT=$NODE_PORT
NODE_PORT=32070
nothing@pc:~$ 
~~~

再进行访问

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

说明服务已经发布成功了


## 使用 labels


Deployment 会为每个 Pod 自动添加上 label

我们可以通过 describe 命令来查看

~~~
nothing@pc:~$ kubectl describe deployment
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Tue, 13 Aug 2019 22:29:02 +0800
Labels:                 run=kubernetes-bootcamp
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               run=kubernetes-bootcamp
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
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
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   kubernetes-bootcamp-69fff89cb6 (1/1 replicas created)
Events:          <none>
nothing@pc:~$
~~~

## 通过标签来查看 pod

通过加上 **`-l`** 参数来加上标签的限定

~~~
nothing@pc:~$ kubectl get pods -l run=kubernetes-bootcamp
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fff89cb6-42cpz   1/1     Running   1          13h
nothing@pc:~$ 
~~~

## 通过标签来查看 services 

~~~
nothing@pc:~$ kubectl get services -l run=kubernetes-bootcamp
NAME                  TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes-bootcamp   NodePort   10.98.252.197   <none>        80:32070/TCP   25m
nothing@pc:~$
~~~


## 给 Pod 打标签

获取 Pod 名存到变量中

~~~
nothing@pc:~$ export POD_NAME=$(kubectl get pods -o go-template --template '{\{range .items}\}{\{.metadata.name}\}{\{"\n"}\}{\{end}\}')
nothing@pc:~$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-69fff89cb6-42cpz
nothing@pc:~$ 
~~~

给 pod 添加一个新标签

使用 **`label`** 命令加上对象来给对象加上新标签

~~~
nothing@pc:~$ kubectl label pod $POD_NAME app=v1
pod/kubernetes-bootcamp-69fff89cb6-42cpz labeled
nothing@pc:~$ 
~~~

## 查看 pod 的标签

使用 describe 来查看新的标签

~~~
nothing@pc:~$ kubectl describe pods $POD_NAME
Name:           kubernetes-bootcamp-69fff89cb6-42cpz
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Tue, 13 Aug 2019 22:29:02 +0800
Labels:         app=v1
                pod-template-hash=69fff89cb6
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.5
Controlled By:  ReplicaSet/kubernetes-bootcamp-69fff89cb6
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://f78351afa88831b68bc0fe4fe766cb6491de29b5caef97e903d4013deea1aab7
    Image:          daocloud.io/library/nginx
    Image ID:       docker-pullable://daocloud.io/library/nginx@sha256:f83b2ffd963ac911f9e638184c8d580cc1f3139d5c8c33c87c3fb90aebdebf76
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 14 Aug 2019 09:01:26 +0800
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 13 Aug 2019 22:30:52 +0800
      Finished:     Wed, 14 Aug 2019 00:36:33 +0800
    Ready:          True
    Restart Count:  1
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-hsc6d (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-hsc6d:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-hsc6d
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:          <none>
nothing@pc:~$ 
~~~


## 删除一个 service

通过 delete 命令来删除一个 service

可以使用 label 来进行限定

~~~
nothing@pc:~$ kubectl delete service -l run=kubernetes-bootcamp
service "kubernetes-bootcamp" deleted
nothing@pc:~$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   15h
nothing@pc:~$ 
~~~

可以进行访问来确认服务是否删除

~~~
nothing@pc:~$ curl $(minikube ip):$NODE_PORT
curl: (7) Failed to connect to 192.168.99.100 port 32070: 拒绝连接
nothing@pc:~$ 
~~~

这就说明，应用不再能被外界访问了

但是依旧可以从内部进行访问

~~~
nothing@pc:~$ kubectl exec -ti $POD_NAME curl localhost:80
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

>**Note:** 运行这些命令的前提是容器中已经安装有 curl 命令

---

# 总结

主要是 kubectl 命令的使用

~~~
kubectl describe deployment
kubectl get pods -l run=kubernetes-bootcamp
kubectl get services -l run=kubernetes-bootcamp
export POD_NAME=$(kubectl get pods -o go-template --template '{\{range .items}\}{\{.metadata.name}\}{\{"\n"}\}{\{end}\}')
echo Name of the Pod: $POD_NAME
kubectl label pod $POD_NAME app=v1
kubectl describe pods $POD_NAME
kubectl describe pods -l app=v1
kubectl get service
kubectl get services
kubectl get pods
kubectl get pod
kubectl delete service -l run=kubernetes-bootcamp
curl $(minikube ip):$NODE_PORT
kubectl exec -it $POD_NAME bash
kubectl exec -ti $POD_NAME curl localhost:80
~~~

* TOC
{:toc}

---


[expose_interactive]:https://kubernetes.io/docs/tutorials/kubernetes-basics/expose/expose-interactive/
[kubernetes]:https://kubernetes.io/


