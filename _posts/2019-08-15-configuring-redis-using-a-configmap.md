---
layout: post
title: "Configuring Redis Using a Configmap"
date: 2019-08-15 10:30:35
image: '/assets/img/'
description: '使用 ConfigMap 来配置 Redis'
main-class:  'kubernetes'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
 - kubernetes
twitter_text:  'configuring redis using a configmap'
introduction: 'configuring redis using a configmap'
---


# 前言


**[k8s][kubernetes]** 中 **ConfigMaps** 可以让应用与配置解偶，从而让应用具备更好的移植性

>ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized applications portable

这篇讲如何在 **[k8s][kubernetes]** 中应用 **ConfigMaps** 来配置 Redis

参考 **[Configuring Redis using a ConfigMap][configure_redis_using_configmap]** 


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

## 创建配置文件

创建 redis 的配置

~~~
nothing@pc:/tmp/redis$ vim redis-config 
nothing@pc:/tmp/redis$ cat redis-config 
maxmemory 2mb
maxmemory-policy allkeys-lru
nothing@pc:/tmp/redis$ 
~~~

创建 pods 的配置

~~~
nothing@pc:/tmp/redis$ cat redis-pod.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: redis
spec:
  containers:
  - name: redis
    image: redis:5.0.4  ## 指定镜像
    command:
      - redis-server
      - "/redis-master/redis.conf"   ## 这里指定了配置的路径
    env:
    - name: MASTER
      value: "true"
    ports:
    - containerPort: 6379
    resources:
      limits:
        cpu: "0.1"
    volumeMounts:
    - mountPath: /redis-master-data  ## 创建数据存放路径
      name: data
    - mountPath: /redis-master   ## 创建配置存放路径
      name: config
  volumes:
    - name: data
      emptyDir: {}   ## 指定一个空目录
    - name: config
      configMap:
        name: example-redis-config  ## 使用 configmap
        items:
        - key: redis-config ## 使用 redis-config 中的值作为内容
          path: redis.conf   ## 路径指定为 redis.conf
nothing@pc:/tmp/redis$ 
~~~

创建总配置

~~~
nothing@pc:/tmp/redis$ vim kustomization.yaml 
nothing@pc:/tmp/redis$ cat kustomization.yaml 
configMapGenerator:
- name: example-redis-config
  files:
  - redis-config  ## 通过文件来创建 configmap
resources:
- redis-pod.yaml  ## 直接引用 pod 的配置进行创建
nothing@pc:/tmp/redis$
~~~

## 创建项目


~~~
nothing@pc:/tmp/redis$ kubectl  get deployments
No resources found.
nothing@pc:/tmp/redis$ kubectl  get pods
No resources found.
nothing@pc:/tmp/redis$ kubectl  get configmap
No resources found.
nothing@pc:/tmp/redis$ 
~~~

直接使用 **`apply`** 来创建

~~~
nothing@pc:/tmp/redis$ ls
kustomization.yaml  redis-config  redis-pod.yaml
nothing@pc:/tmp/redis$ 
nothing@pc:/tmp/redis$ kubectl apply -k . 
configmap/example-redis-config-dgh9dg555m created
pod/redis created
nothing@pc:/tmp/redis$ kubectl  get configmap
NAME                              DATA   AGE
example-redis-config-dgh9dg555m   1      4s
nothing@pc:/tmp/redis$ kubectl  get configmap -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    redis-config: |
      maxmemory 2mb
      maxmemory-policy allkeys-lru
  kind: ConfigMap
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"v1","data":{"redis-config":"maxmemory 2mb\nmaxmemory-policy allkeys-lru\n"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"example-redis-config-dgh9dg555m","namespace":"default"}}
    creationTimestamp: "2019-08-15T15:26:06Z"
    name: example-redis-config-dgh9dg555m
    namespace: default
    resourceVersion: "156279"
    selfLink: /api/v1/namespaces/default/configmaps/example-redis-config-dgh9dg555m
    uid: 4899a831-0077-4285-9b8a-6e207d6b9784
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
nothing@pc:/tmp/redis$ kubectl  get deployments
No resources found.
nothing@pc:/tmp/redis$ kubectl  get pods
NAME    READY   STATUS    RESTARTS   AGE
redis   1/1     Running   0          27s
nothing@pc:/tmp/redis$ kubectl  describe  pods/redis
Name:         redis
Namespace:    default
Priority:     0
Node:         minikube/10.0.2.15
Start Time:   Thu, 15 Aug 2019 23:26:06 +0800
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration:
                {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"name":"redis","namespace":"default"},"spec":{"containers":[{"command":["redi...
Status:       Running
IP:           172.17.0.4
Containers:
  redis:
    Container ID:  docker://9401acf35362a61faa9dee58d7994cfd6faacf68c8b561396b1db9d6e56ad3ae
    Image:         redis:5.0.4
    Image ID:      docker-pullable://daocloud.io/library/redis@sha256:82ac0e8f4f2cb5db18714b726febea6de9666ad9d9ad6f62f433f073bc3048f0
    Port:          6379/TCP
    Host Port:     0/TCP
    Command:
      redis-server
      /redis-master/redis.conf
    State:          Running
      Started:      Thu, 15 Aug 2019 23:26:07 +0800
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:  100m
    Requests:
      cpu:  100m
    Environment:
      MASTER:  true
    Mounts:
      /redis-master from config (rw)
      /redis-master-data from data (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-hsc6d (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  config:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      example-redis-config-dgh9dg555m
    Optional:  false
  default-token-hsc6d:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-hsc6d
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  48s   default-scheduler  Successfully assigned default/redis to minikube
  Normal  Pulled     47s   kubelet, minikube  Container image "redis:5.0.4" already present on machine
  Normal  Created    47s   kubelet, minikube  Created container redis
  Normal  Started    47s   kubelet, minikube  Started container redis
nothing@pc:/tmp/redis$ 
~~~

## 访问实例

~~~
nothing@pc:/tmp/redis$ kubectl exec -it redis redis-cli
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "2097152"
127.0.0.1:6379> config get maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"
127.0.0.1:6379> 
127.0.0.1:6379> exit
nothing@pc:/tmp/redis$ 
~~~

## 进入容器查看

~~~
nothing@pc:/tmp/redis$ kubectl exec -it redis bash
root@redis:/data# cat /redis-master/redis.conf 
maxmemory 2mb
maxmemory-policy allkeys-lru
root@redis:/data# ls -la /redis-master-data/
total 8
drwxrwxrwx 2 root root 4096 Aug 15 15:26 .
drwxr-xr-x 1 root root 4096 Aug 15 15:26 ..
root@redis:/data# ls -la /redis-master/     
total 12
drwxrwxrwx 3 root root 4096 Aug 15 15:26 .
drwxr-xr-x 1 root root 4096 Aug 15 15:26 ..
drwxr-xr-x 2 root root 4096 Aug 15 15:26 ..2019_08_15_15_26_06.697347756
lrwxrwxrwx 1 root root   31 Aug 15 15:26 ..data -> ..2019_08_15_15_26_06.697347756
lrwxrwxrwx 1 root root   17 Aug 15 15:26 redis.conf -> ..data/redis.conf
root@redis:/data# ls -la /redis-master/..2019_08_15_15_26_06.697347756
total 12
drwxr-xr-x 2 root root 4096 Aug 15 15:26 .
drwxrwxrwx 3 root root 4096 Aug 15 15:26 ..
-rw-r--r-- 1 root root   43 Aug 15 15:26 redis.conf
root@redis:/data# cat /redis-master/..2019_08_15_15_26_06.697347756/redis.conf 
maxmemory 2mb
maxmemory-policy allkeys-lru
root@redis:/data#
root@redis:/data# redis-cli 
127.0.0.1:6379> config get maxmemory
1) "maxmemory"
2) "2097152"
127.0.0.1:6379> config get maxmemory-policy
1) "maxmemory-policy"
2) "allkeys-lru"
127.0.0.1:6379> 
~~~

可知，其实这都是软链接



---

# 总结

主要是 kubectl 命令的使用

通过 pod 结合 configmap 的方便，可以极大的将基础环境文本化

从而有效提升管理效率

~~~
kubectl apply -k . 
kubectl  get configmap -o yaml
kubectl  get deployments
kubectl  get pods
kubectl  describe  pods/redis
kubectl exec -it redis redis-cli
kubectl exec -it redis bash
~~~


* TOC
{:toc}

---


[configure_redis_using_configmap]:https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/
[kubernetes]:https://kubernetes.io/


