---
layout: post
title: "Exposing an External IP Address"
date: 2019-08-16 02:40:12
image: '/assets/img/'
description: '发布外网IP来访问集群应用'
main-class:  'kubernetes'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
 - kubernetes
twitter_text:  'exposing an external IP address to access an application in a cluster'
introduction: 'exposing an external IP address to access an application in a cluster'
---

# 前言


**[k8s][kubernetes]** 中 **Service** 可以让流量与应用解偶，从而让应用具备更好的灵活性

这篇讲如何在 **[k8s][kubernetes]** 中使用 **Service** 来发布应用

参考 **[Exposing an External IP Address to Access an Application in a Cluster][expose_external_ip_address]** 


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

## 构建一个镜像


这里使用 nodejs 构建一个简单的 web 应用

~~~
nothing@pc:~$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ mkdir hello ; cd hello
$ vi server.js  
$ cat server.js 
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.write(process.env.HOSTNAME);
  response.end(" Hello World!  |v1| \n");
};
var www = http.createServer(handleRequest);
www.listen(8080);
$ vi Dockerfile 
$ cat Dockerfile 
FROM node:6.14.2
EXPOSE 8080
COPY server.js .
CMD node server.js
$ docker build -t   registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1  .
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM node:6.14.2
 ---> 00165cd5d0c0
Step 2/4 : EXPOSE 8080
 ---> Using cache
 ---> e23583b93070
Step 3/4 : COPY server.js .
 ---> Using cache
 ---> 8ae53b81b855
Step 4/4 : CMD node server.js
 ---> Using cache
 ---> 7da93b374b56
Successfully built 7da93b374b56
Successfully tagged registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1
$ 
~~~

## 创建应用配置

~~~
nothing@pc:/tmp$ mkdir lb
nothing@pc:/tmp$ cd lb
nothing@pc:/tmp/lb$ vim load-balancer-example.yaml
nothing@pc:/tmp/lb$ cat load-balancer-example.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: load-balancer-example
  name: hello-world
spec:
  replicas: 5
  selector:
    matchLabels:
      app.kubernetes.io/name: load-balancer-example
  template:
    metadata:
      labels:
        app.kubernetes.io/name: load-balancer-example
    spec:
      containers:
      - image: registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1
        name: hello-world
        ports:
        - containerPort: 8080
nothing@pc:/tmp/lb$
~~~

## 应用配置文件生成应用

先检查当前环境中的应用状态

~~~
nothing@pc:/tmp/lb$ kubectl get deployment
No resources found.
nothing@pc:/tmp/lb$ 
nothing@pc:/tmp/lb$ kubectl get pods
No resources found.
nothing@pc:/tmp/lb$ 
nothing@pc:/tmp/lb$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d16h
nothing@pc:/tmp/lb$ 
~~~

应用配置生成实例

~~~
nothing@pc:/tmp/lb$ kubectl apply -f load-balancer-example.yaml 
deployment.apps/hello-world created
nothing@pc:/tmp/lb$ kubectl get deployments
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hello-world   5/5     5            5           6s
nothing@pc:/tmp/lb$ kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   2d15h
nothing@pc:/tmp/lb$ kubectl get pods
NAME                           READY   STATUS    RESTARTS   AGE
hello-world-7468b7fd8f-5lbxj   1/1     Running   0          18s
hello-world-7468b7fd8f-dpfpc   1/1     Running   0          18s
hello-world-7468b7fd8f-k5pbc   1/1     Running   0          18s
hello-world-7468b7fd8f-m6qlh   1/1     Running   0          18s
hello-world-7468b7fd8f-z744h   1/1     Running   0          18s
nothing@pc:/tmp/lb$ kubectl get replicasets
NAME                     DESIRED   CURRENT   READY   AGE
hello-world-7468b7fd8f   5         5         5       44m
nothing@pc:/tmp/lb$ 
~~~

前面的命令创建了一个 **[Deployment ][deployment]**  然后关联了一个 **[ReplicaSet][replicaset]** 

这个 **[ReplicaSet][replicaset]** 有五个 **[Pods][pod]**

这五个 **[Pods][pod]** 都运行着相同的实例

## 查看 Deployment 的信息

~~~
nothing@pc:/tmp/lb$ kubectl get deployments hello-world
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
hello-world   5/5     5            5           141m
nothing@pc:/tmp/lb$ kubectl describe deployments hello-world
Name:                   hello-world
Namespace:              default
CreationTimestamp:      Fri, 16 Aug 2019 11:48:36 +0800
Labels:                 app.kubernetes.io/name=load-balancer-example
Annotations:            deployment.kubernetes.io/revision: 1
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"labels":{"app.kubernetes.io/name":"load-balancer-example"},"name...
Selector:               app.kubernetes.io/name=load-balancer-example
Replicas:               5 desired | 5 updated | 5 total | 5 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app.kubernetes.io/name=load-balancer-example
  Containers:
   hello-world:
    Image:        registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1
    Port:         8080/TCP
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
NewReplicaSet:   hello-world-7468b7fd8f (5/5 replicas created)
Events:          <none>
nothing@pc:/tmp/lb$ 
~~~

## 查看 ReplicaSet 的信息

~~~
nothing@pc:/tmp/lb$ kubectl get replicasets
NAME                     DESIRED   CURRENT   READY   AGE
hello-world-7468b7fd8f   5         5         5       142m
nothing@pc:/tmp/lb$ kubectl  describe replicasets
Name:           hello-world-7468b7fd8f
Namespace:      default
Selector:       app.kubernetes.io/name=load-balancer-example,pod-template-hash=7468b7fd8f
Labels:         app.kubernetes.io/name=load-balancer-example
                pod-template-hash=7468b7fd8f
Annotations:    deployment.kubernetes.io/desired-replicas: 5
                deployment.kubernetes.io/max-replicas: 7
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/hello-world
Replicas:       5 current / 5 desired
Pods Status:    5 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app.kubernetes.io/name=load-balancer-example
           pod-template-hash=7468b7fd8f
  Containers:
   hello-world:
    Image:        registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:           <none>
nothing@pc:/tmp/lb$ 
~~~

## 创建 service 对象

~~~
nothing@pc:/tmp/lb$ kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
service/my-service exposed
nothing@pc:/tmp/lb$ 
~~~

## 查看 service 对象

~~~
nothing@pc:/tmp/lb$ kubectl get services my-service
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
my-service   LoadBalancer   10.111.36.143   <pending>     8080:31691/TCP   4m45s
nothing@pc:/tmp/lb$ kubectl  describe  services my-service
Name:                     my-service
Namespace:                default
Labels:                   app.kubernetes.io/name=load-balancer-example
Annotations:              <none>
Selector:                 app.kubernetes.io/name=load-balancer-example
Type:                     LoadBalancer
IP:                       10.111.36.143
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  31691/TCP
Endpoints:                172.17.0.3:8080,172.17.0.6:8080,172.17.0.7:8080 + 2 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
nothing@pc:/tmp/lb$
~~~

>**Note:** 由于当前 minikube 并不支持 **LoadBalancer** 的方式，所以还无法为其分配一个外部的 LB VIP，但是可以使用 **NodePort** 的方式来进行访问

## 查看 pods

~~~
nothing@pc:/tmp/lb$ kubectl get pods --output=wide
NAME                           READY   STATUS    RESTARTS   AGE    IP           NODE       NOMINATED NODE   READINESS GATES
hello-world-7468b7fd8f-5lbxj   1/1     Running   0          160m   172.17.0.7   minikube   <none>           <none>
hello-world-7468b7fd8f-dpfpc   1/1     Running   0          160m   172.17.0.8   minikube   <none>           <none>
hello-world-7468b7fd8f-k5pbc   1/1     Running   0          160m   172.17.0.6   minikube   <none>           <none>
hello-world-7468b7fd8f-m6qlh   1/1     Running   0          160m   172.17.0.9   minikube   <none>           <none>
hello-world-7468b7fd8f-z744h   1/1     Running   0          160m   172.17.0.3   minikube   <none>           <none>
nothing@pc:/tmp/lb$ 
~~~

## 进行访问

由于这里并没有 **LoadBalancer Ingress** 的 **External IP** ,所以我们使用 node IP 来进行访问

~~~
nothing@pc:/tmp/lb$ minikube service my-service --url
http://192.168.99.100:31691
nothing@pc:/tmp/lb$ curl http://192.168.99.100:31691
hello-world-7468b7fd8f-5lbxj Hello World!  |v1| 
nothing@pc:/tmp/lb$ curl http://192.168.99.100:31691
hello-world-7468b7fd8f-5lbxj Hello World!  |v1| 
nothing@pc:/tmp/lb$ curl http://192.168.99.100:31691
hello-world-7468b7fd8f-5lbxj Hello World!  |v1| 
nothing@pc:/tmp/lb$ curl http://192.168.99.100:31691
hello-world-7468b7fd8f-m6qlh Hello World!  |v1| 
nothing@pc:/tmp/lb$ curl http://192.168.99.100:31691
hello-world-7468b7fd8f-z744h Hello World!  |v1| 
nothing@pc:/tmp/lb$ curl http://192.168.99.100:31691
hello-world-7468b7fd8f-m6qlh Hello World!  |v1| 
nothing@pc:/tmp/lb$ curl http://192.168.99.100:31691
hello-world-7468b7fd8f-k5pbc Hello World!  |v1| 
nothing@pc:/tmp/lb$ curl http://192.168.99.100:31691
hello-world-7468b7fd8f-5lbxj Hello World!  |v1| 
nothing@pc:/tmp/lb$ curl http://192.168.99.100:31691
hello-world-7468b7fd8f-z744h Hello World!  |v1| 
nothing@pc:/tmp/lb$ 
~~~

可见，从访问结果来看，也是 hit 到了不同的 pod，是在进行负载均衡

使用 **`minikube service my-service`** 会自动打开浏览器并且对服务进行访问

## 删除服务

~~~
nothing@pc:~$ kubectl get services
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          2d21h
my-service   LoadBalancer   10.111.36.143   <pending>     8080:31691/TCP   4h6m
nothing@pc:~$ kubectl delete services my-service
service "my-service" deleted
nothing@pc:~$
~~~

## 删除 Deployment

~~~
nothing@pc:~$ kubectl delete deployment hello-world
deployment "hello-world" deleted
nothing@pc:~$ 
~~~


---

# 总结

主要是 kubectl 命令的使用

通过 service 可以有效发布应用，并且进行负载均衡

~~~
kubectl apply -f load-balancer-example.yaml 
kubectl get deployments
kubectl get services
kubectl get pods
kubectl get replicasets
kubectl get deployments hello-world
kubectl describe deployments hello-world
kubectl get replicasets
kubectl  describe replicasets
kubectl expose deployment hello-world --type=LoadBalancer --name=my-service
kubectl get services my-service
kubectl  describe  services my-service
kubectl get pods --output=wide
minikube service my-service --url
curl http://192.168.99.100:31691
kubectl delete services my-service
kubectl delete deployment hello-world
~~~


* TOC
{:toc}

---


[expose_external_ip_address]:https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/
[kubernetes]:https://kubernetes.io/
[deployment]:https://kubernetes.io/docs/concepts/workloads/controllers/deployment/
[replicaset]:https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
[pod]:https://kubernetes.io/docs/concepts/workloads/pods/pod/


