---
layout: post
title: "Update the App"
date: 2019-08-14 06:07:44
image: '/assets/img/'
description: '升级与回退应用'
main-class:  'tools'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
- tools
twitter_text:  'update the app'
introduction: 'update the app'
---


# 前言


这篇讲如何使用 kubectl 来升级和回滚应用

>The goal of this scenario is to update a deployed application with kubectl set image and to rollback with the rollout undo command

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

## 创建几个版本的镜像

~~~
nothing@pc:~$ minikube ssh 
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ mkdir hello ; cd hello
$ ls
$ vi Dockerfile
$ cat Dockerfile 
FROM node:6.14.2
EXPOSE 8080
COPY server.js .
CMD node server.js
$ vi server.js
$ cat server.js 
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end("Hello World!  |v1| \n");
};
var www = http.createServer(handleRequest);
www.listen(8080);
$ docker build -t   registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1  . 
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM node:6.14.2
 ---> 00165cd5d0c0
Step 2/4 : EXPOSE 8080
 ---> Running in 2ad87d50b9a0
Removing intermediate container 2ad87d50b9a0
 ---> e23583b93070
Step 3/4 : COPY server.js .
 ---> 93d0d7857c58
Step 4/4 : CMD node server.js
 ---> Running in 82a1e7419728
Removing intermediate container 82a1e7419728
 ---> 5873cf47f1b3
Successfully built 5873cf47f1b3
Successfully tagged registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1
$ vi server.js 
$ docker build -t   registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v2  . 
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM node:6.14.2
 ---> 00165cd5d0c0
Step 2/4 : EXPOSE 8080
 ---> Using cache
 ---> e23583b93070
Step 3/4 : COPY server.js .
 ---> 60fffcdf71b8
Step 4/4 : CMD node server.js
 ---> Running in b2dba8fe9ea9
Removing intermediate container b2dba8fe9ea9
 ---> c0608acdf4ff
Successfully built c0608acdf4ff
Successfully tagged registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v2
$ vi server.js 
$ docker build -t   registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3  . 
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM node:6.14.2
 ---> 00165cd5d0c0
Step 2/4 : EXPOSE 8080
 ---> Using cache
 ---> e23583b93070
Step 3/4 : COPY server.js .
 ---> 5b8703dcad54
Step 4/4 : CMD node server.js
 ---> Running in e3aca42d7c0e
Removing intermediate container e3aca42d7c0e
 ---> 99dbb04e9ef8
Successfully built 99dbb04e9ef8
Successfully tagged registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3
$ 
$ docker images
REPOSITORY                                                                        TAG                 IMAGE ID            CREATED              SIZE
registry.cn-hangzhou.aliyuncs.com/wilmos/k8s                                      v3                  99dbb04e9ef8        About a minute ago   660MB
registry.cn-hangzhou.aliyuncs.com/wilmos/k8s                                      v2                  c0608acdf4ff        About a minute ago   660MB
registry.cn-hangzhou.aliyuncs.com/wilmos/k8s                                      v1                  5873cf47f1b3        About a minute ago   660MB
daocloud.io/library/nginx                                                         v2                  3348d95a6909        3 hours ago          109MB
daocloud.io/nginx                                                                 v2                  3348d95a6909        3 hours ago          109MB
daocloud.io/library/nginx                                                         v1                  bccac6db4e09        3 hours ago          109MB
daocloud.io/nginx                                                                 v1                  bccac6db4e09        3 hours ago          109MB
daocloud.io/library/nginx                                                         latest              98ebf73aba75        3 weeks ago          109MB
daocloud.io/nginx                                                                 latest              98ebf73aba75        3 weeks ago          109MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy                    v1.15.0             d235b23c3570        7 weeks ago          82.4MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver                v1.15.0             201c7a840312        7 weeks ago          207MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager       v1.15.0             8328bb49b652        7 weeks ago          159MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler                v1.15.0             2d3813851e87        7 weeks ago          81.1MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-addon-manager            v9.0                119701e77cbc        6 months ago         83.1MB
registry.cn-hangzhou.aliyuncs.com/google_containers/coredns                       1.3.1               eb516548c180        7 months ago         40.3MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64    v1.10.1             f9aed6605b81        8 months ago         122MB
registry.cn-hangzhou.aliyuncs.com/google_containers/etcd                          3.3.10              2c4adeb21b4f        8 months ago         258MB
registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-sidecar-amd64         1.14.13             4b2e93f0133d        10 months ago        42.9MB
registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-kube-dns-amd64        1.14.13             55a3c5209c5e        10 months ago        51.2MB
registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-dnsmasq-nanny-amd64   1.14.13             6dc8ef8287d3        10 months ago        41.4MB
node                                                                              6.14.2              00165cd5d0c0        14 months ago        660MB
registry.cn-hangzhou.aliyuncs.com/google_containers/pause                         3.1                 da86e6ba6ca1        20 months ago        742kB
registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner           v1.8.1              4689081edb10        21 months ago        80.8MB
$ 
~~~

## 构建一个 Deployment 并且进行发布

~~~
nothing@pc:~$ kubectl get deployment
No resources found.
nothing@pc:~$ kubectl run kubernetes-bootcamp --image=registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1 --port=8080  --replicas=4
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/kubernetes-bootcamp created
nothing@pc:~$ kubectl get deployment
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           3s
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5b7588689d-khkhs   1/1     Running   0          7s
kubernetes-bootcamp-5b7588689d-m6zp4   1/1     Running   0          7s
kubernetes-bootcamp-5b7588689d-sw4zk   1/1     Running   0          7s
kubernetes-bootcamp-5b7588689d-w7v8w   1/1     Running   0          7s
nothing@pc:~$ kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port=8080
service/kubernetes-bootcamp exposed
nothing@pc:~$ minikube service kubernetes-bootcamp --url
http://192.168.99.100:32306
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v1| 
nothing@pc:~$ 
~~~

## 查看 pod 的信息

~~~
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-5b7588689d-khkhs   1/1     Running   0          3m3s
kubernetes-bootcamp-5b7588689d-m6zp4   1/1     Running   0          3m3s
kubernetes-bootcamp-5b7588689d-sw4zk   1/1     Running   0          3m3s
kubernetes-bootcamp-5b7588689d-w7v8w   1/1     Running   0          3m3s
nothing@pc:~$
~~~

如果要看 Pod 的镜像版本，通过 describe

~~~
nothing@pc:~$ kubectl describe pods
Name:           kubernetes-bootcamp-5b7588689d-khkhs
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 14 Aug 2019 17:17:19 +0800
Labels:         pod-template-hash=5b7588689d
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.8
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b7588689d
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://e5f276ecbb717e161c6d89f6df8c8727bb991c9c35d43ed366417b77aecd76a5
    Image:          registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1
    Image ID:       docker://sha256:5873cf47f1b377215d4c7e2b5f91ed77b072b747bd9a40cfe092bb41badfafc1
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 14 Aug 2019 17:17:20 +0800
    Ready:          True
    Restart Count:  0
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
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b7588689d-khkhs to minikube
  Normal  Pulled     10m   kubelet, minikube  Container image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1" already present on machine
  Normal  Created    10m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    10m   kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-5b7588689d-m6zp4
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 14 Aug 2019 17:17:19 +0800
Labels:         pod-template-hash=5b7588689d
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.7
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b7588689d
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://b01b938011445013d562c555030053cd6b71aeaf0518d9e4ea1ec8915632a02b
    Image:          registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1
    Image ID:       docker://sha256:5873cf47f1b377215d4c7e2b5f91ed77b072b747bd9a40cfe092bb41badfafc1
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 14 Aug 2019 17:17:20 +0800
    Ready:          True
    Restart Count:  0
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
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b7588689d-m6zp4 to minikube
  Normal  Pulled     10m   kubelet, minikube  Container image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1" already present on machine
  Normal  Created    10m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    10m   kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-5b7588689d-sw4zk
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 14 Aug 2019 17:17:19 +0800
Labels:         pod-template-hash=5b7588689d
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.6
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b7588689d
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://ef5bfd79d6a8b1576a6132352fddd21385a59bdacbee974a1364d0976751758d
    Image:          registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1
    Image ID:       docker://sha256:5873cf47f1b377215d4c7e2b5f91ed77b072b747bd9a40cfe092bb41badfafc1
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 14 Aug 2019 17:17:20 +0800
    Ready:          True
    Restart Count:  0
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
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b7588689d-sw4zk to minikube
  Normal  Pulled     10m   kubelet, minikube  Container image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1" already present on machine
  Normal  Created    10m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    10m   kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-5b7588689d-w7v8w
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 14 Aug 2019 17:17:19 +0800
Labels:         pod-template-hash=5b7588689d
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.5
Controlled By:  ReplicaSet/kubernetes-bootcamp-5b7588689d
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://b10f824893413113d39d8b84e727272ffe57ab72a4550745704b5e0eae9276ab
    Image:          registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1
    Image ID:       docker://sha256:5873cf47f1b377215d4c7e2b5f91ed77b072b747bd9a40cfe092bb41badfafc1
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 14 Aug 2019 17:17:20 +0800
    Ready:          True
    Restart Count:  0
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
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  10m   default-scheduler  Successfully assigned default/kubernetes-bootcamp-5b7588689d-w7v8w to minikube
  Normal  Pulled     10m   kubelet, minikube  Container image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1" already present on machine
  Normal  Created    10m   kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    10m   kubelet, minikube  Started container kubernetes-bootcamp
nothing@pc:~$ 
~~~


## 升级镜像

~~~
nothing@pc:~$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v2 
deployment.extensions/kubernetes-bootcamp image updated
nothing@pc:~$ kubectl get deployment
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           13m
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-58b56b5ff9-8qw66   1/1     Running       0          12s
kubernetes-bootcamp-58b56b5ff9-gb6kf   1/1     Running       0          12s
kubernetes-bootcamp-58b56b5ff9-ppg2n   1/1     Running       0          9s
kubernetes-bootcamp-58b56b5ff9-rg2tj   1/1     Running       0          10s
kubernetes-bootcamp-5b7588689d-khkhs   1/1     Terminating   0          13m
kubernetes-bootcamp-5b7588689d-m6zp4   1/1     Terminating   0          13m
kubernetes-bootcamp-5b7588689d-sw4zk   1/1     Terminating   0          13m
kubernetes-bootcamp-5b7588689d-w7v8w   1/1     Terminating   0          13m
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v2| 
nothing@pc:~$
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-58b56b5ff9-8qw66   1/1     Running   0          67s
kubernetes-bootcamp-58b56b5ff9-gb6kf   1/1     Running   0          67s
kubernetes-bootcamp-58b56b5ff9-ppg2n   1/1     Running   0          64s
kubernetes-bootcamp-58b56b5ff9-rg2tj   1/1     Running   0          65s
nothing@pc:~$
~~~

使用 **`set image`** 命令来替换镜像

这个过程会创建新的 Pod 然后删除旧版本的 Pod

~~~
nothing@pc:~$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3 
deployment.extensions/kubernetes-bootcamp image updated
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS              RESTARTS   AGE
kubernetes-bootcamp-57cb7dcc4d-8nb6g   1/1     Running             0          2s
kubernetes-bootcamp-57cb7dcc4d-bm66b   0/1     Pending             0          0s
kubernetes-bootcamp-57cb7dcc4d-kggh6   1/1     Running             0          2s
kubernetes-bootcamp-57cb7dcc4d-v4nkl   0/1     ContainerCreating   0          0s
kubernetes-bootcamp-58b56b5ff9-8qw66   1/1     Terminating         0          3m40s
kubernetes-bootcamp-58b56b5ff9-gb6kf   1/1     Running             0          3m40s
kubernetes-bootcamp-58b56b5ff9-ppg2n   1/1     Terminating         0          3m37s
kubernetes-bootcamp-58b56b5ff9-rg2tj   1/1     Terminating         0          3m38s
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v3| 
nothing@pc:~$curl http://192.168.99.100:32306
Hello World!  |v3| 
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-57cb7dcc4d-8nb6g   1/1     Running       0          14s
kubernetes-bootcamp-57cb7dcc4d-bm66b   1/1     Running       0          12s
kubernetes-bootcamp-57cb7dcc4d-kggh6   1/1     Running       0          14s
kubernetes-bootcamp-57cb7dcc4d-v4nkl   1/1     Running       0          12s
kubernetes-bootcamp-58b56b5ff9-8qw66   1/1     Terminating   0          3m52s
kubernetes-bootcamp-58b56b5ff9-gb6kf   1/1     Terminating   0          3m52s
kubernetes-bootcamp-58b56b5ff9-ppg2n   1/1     Terminating   0          3m49s
kubernetes-bootcamp-58b56b5ff9-rg2tj   1/1     Terminating   0          3m50s
nothing@pc:~$
~~~

## 通过 service 信息来看发布的端口

~~~
nothing@pc:~$ kubectl describe service/kubernetes-bootcamp
Name:                     kubernetes-bootcamp
Namespace:                default
Labels:                   run=kubernetes-bootcamp
Annotations:              <none>
Selector:                 run=kubernetes-bootcamp
Type:                     NodePort
IP:                       10.101.119.17
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  32306/TCP
Endpoints:                172.17.0.5:8080,172.17.0.6:8080,172.17.0.7:8080 + 1 more...
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
nothing@pc:~$
~~~

## 进行访问

~~~
nothing@pc:~$ export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{\{(index .spec.ports 0).nodePort}\}')
nothing@pc:~$ echo NODE_PORT=$NODE_PORT
NODE_PORT=32306
nothing@pc:~$ curl $(minikube ip):$NODE_PORT
Hello World!  |v3| 
nothing@pc:~$ curl $(minikube ip):$NODE_PORT
Hello World!  |v3| 
nothing@pc:~$ 
~~~

## 查看升级状态

~~~
nothing@pc:~$ kubectl rollout status deployments/kubernetes-bootcamp
deployment "kubernetes-bootcamp" successfully rolled out
nothing@pc:~$
~~~

通过查看 pods 的详细信息来进一步确认状态 

~~~
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-57cb7dcc4d-8nb6g   1/1     Running   0          5m1s
kubernetes-bootcamp-57cb7dcc4d-bm66b   1/1     Running   0          4m59s
kubernetes-bootcamp-57cb7dcc4d-kggh6   1/1     Running   0          5m1s
kubernetes-bootcamp-57cb7dcc4d-v4nkl   1/1     Running   0          4m59s
nothing@pc:~$ kubectl describe pods
Name:           kubernetes-bootcamp-57cb7dcc4d-8nb6g
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 14 Aug 2019 17:33:54 +0800
Labels:         pod-template-hash=57cb7dcc4d
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.5
Controlled By:  ReplicaSet/kubernetes-bootcamp-57cb7dcc4d
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://4b728b9fe52e66b7662771b9b86929b7fb4dd24b951b5bbd4722c96036d5b194
    Image:          registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3
    Image ID:       docker://sha256:99dbb04e9ef82cd72b32fbee2593ba06e7a41745abcc41a301cb17ed9840053b
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 14 Aug 2019 17:33:55 +0800
    Ready:          True
    Restart Count:  0
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
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  5m8s  default-scheduler  Successfully assigned default/kubernetes-bootcamp-57cb7dcc4d-8nb6g to minikube
  Normal  Pulled     5m7s  kubelet, minikube  Container image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3" already present on machine
  Normal  Created    5m7s  kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    5m7s  kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-57cb7dcc4d-bm66b
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 14 Aug 2019 17:33:56 +0800
Labels:         pod-template-hash=57cb7dcc4d
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.8
Controlled By:  ReplicaSet/kubernetes-bootcamp-57cb7dcc4d
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://e690ab179ff9b60a0c6165bba9036c39cb3ec694d08924046ee97749377c650c
    Image:          registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3
    Image ID:       docker://sha256:99dbb04e9ef82cd72b32fbee2593ba06e7a41745abcc41a301cb17ed9840053b
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 14 Aug 2019 17:33:57 +0800
    Ready:          True
    Restart Count:  0
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
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  5m6s  default-scheduler  Successfully assigned default/kubernetes-bootcamp-57cb7dcc4d-bm66b to minikube
  Normal  Pulled     5m5s  kubelet, minikube  Container image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3" already present on machine
  Normal  Created    5m5s  kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    5m5s  kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-57cb7dcc4d-kggh6
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 14 Aug 2019 17:33:54 +0800
Labels:         pod-template-hash=57cb7dcc4d
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.6
Controlled By:  ReplicaSet/kubernetes-bootcamp-57cb7dcc4d
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://9fa2e060d2042d206148a72f2066ba4428794f269f645a5f1e6970d31acd8e78
    Image:          registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3
    Image ID:       docker://sha256:99dbb04e9ef82cd72b32fbee2593ba06e7a41745abcc41a301cb17ed9840053b
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 14 Aug 2019 17:33:55 +0800
    Ready:          True
    Restart Count:  0
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
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  5m8s  default-scheduler  Successfully assigned default/kubernetes-bootcamp-57cb7dcc4d-kggh6 to minikube
  Normal  Pulled     5m7s  kubelet, minikube  Container image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3" already present on machine
  Normal  Created    5m7s  kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    5m7s  kubelet, minikube  Started container kubernetes-bootcamp


Name:           kubernetes-bootcamp-57cb7dcc4d-v4nkl
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 14 Aug 2019 17:33:56 +0800
Labels:         pod-template-hash=57cb7dcc4d
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.7
Controlled By:  ReplicaSet/kubernetes-bootcamp-57cb7dcc4d
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://3db253105ddcaaba996ca55cbff2ba75c5a36986960de1b34a810a1ca8b14d1f
    Image:          registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3
    Image ID:       docker://sha256:99dbb04e9ef82cd72b32fbee2593ba06e7a41745abcc41a301cb17ed9840053b
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Wed, 14 Aug 2019 17:33:57 +0800
    Ready:          True
    Restart Count:  0
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
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  5m6s  default-scheduler  Successfully assigned default/kubernetes-bootcamp-57cb7dcc4d-v4nkl to minikube
  Normal  Pulled     5m5s  kubelet, minikube  Container image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3" already present on machine
  Normal  Created    5m5s  kubelet, minikube  Created container kubernetes-bootcamp
  Normal  Started    5m5s  kubelet, minikube  Started container kubernetes-bootcamp
nothing@pc:~$ 
~~~

## 执行一个错误升级


~~~
nothing@pc:~$ kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v10 
deployment.extensions/kubernetes-bootcamp image updated
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   3/4     2            3           23m
nothing@pc:~$
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v3| 
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v3| 
nothing@pc:~$
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS         RESTARTS   AGE
kubernetes-bootcamp-559647b9d-hqvmc    0/1     ErrImagePull   0          37s
kubernetes-bootcamp-559647b9d-jjr5f    0/1     ErrImagePull   0          37s
kubernetes-bootcamp-57cb7dcc4d-8nb6g   1/1     Running        0          7m46s
kubernetes-bootcamp-57cb7dcc4d-bm66b   1/1     Running        0          7m44s
kubernetes-bootcamp-57cb7dcc4d-kggh6   1/1     Running        0          7m46s
kubernetes-bootcamp-57cb7dcc4d-v4nkl   0/1     Terminating    0          7m44s
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS             RESTARTS   AGE
kubernetes-bootcamp-559647b9d-hqvmc    0/1     ImagePullBackOff   0          52s
kubernetes-bootcamp-559647b9d-jjr5f    0/1     ImagePullBackOff   0          52s
kubernetes-bootcamp-57cb7dcc4d-8nb6g   1/1     Running            0          8m1s
kubernetes-bootcamp-57cb7dcc4d-bm66b   1/1     Running            0          7m59s
kubernetes-bootcamp-57cb7dcc4d-kggh6   1/1     Running            0          8m1s
nothing@pc:~$ 
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   3/4     2            3           25m
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v3| 
nothing@pc:~$ 
~~~

之所以错误，是因为系统中并没有 v10 版本的镜像

同时可以看到在镜像错误的情况下，并没有影响到应用的可用性

可以通过 **`describe`** 来看到更详细的关于 pod 的错误信息

## 进行回退

~~~
nothing@pc:~$ kubectl rollout undo deployments/kubernetes-bootcamp
deployment.extensions/kubernetes-bootcamp rolled back
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           32m
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-57cb7dcc4d-8nb6g   1/1     Running   0          15m
kubernetes-bootcamp-57cb7dcc4d-bm66b   1/1     Running   0          15m
kubernetes-bootcamp-57cb7dcc4d-kggh6   1/1     Running   0          15m
kubernetes-bootcamp-57cb7dcc4d-smhmv   1/1     Running   0          10s
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v3| 
nothing@pc:~$
~~~

使用 **`rollout undo`** 可以回退到之前的版本



~~~
nothing@pc:~$ kubectl rollout undo deployments/kubernetes-bootcamp
deployment.extensions/kubernetes-bootcamp rolled back
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   3/4     2            3           34m
nothing@pc:~$ 
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS             RESTARTS   AGE
kubernetes-bootcamp-559647b9d-47stc    0/1     ImagePullBackOff   0          22s
kubernetes-bootcamp-559647b9d-4fvrh    0/1     ImagePullBackOff   0          22s
kubernetes-bootcamp-57cb7dcc4d-8nb6g   1/1     Running            0          18m
kubernetes-bootcamp-57cb7dcc4d-bm66b   1/1     Running            0          18m
kubernetes-bootcamp-57cb7dcc4d-kggh6   1/1     Running            0          18m
kubernetes-bootcamp-57cb7dcc4d-smhmv   1/1     Terminating        0          2m35s
nothing@pc:~$ 
nothing@pc:~$ kubectl describe pods/kubernetes-bootcamp-559647b9d-47stc
Name:           kubernetes-bootcamp-559647b9d-47stc
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Wed, 14 Aug 2019 17:51:50 +0800
Labels:         pod-template-hash=559647b9d
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Pending
IP:             172.17.0.10
Controlled By:  ReplicaSet/kubernetes-bootcamp-559647b9d
Containers:
  kubernetes-bootcamp:
    Container ID:   
    Image:          registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v10
    Image ID:       
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Waiting
      Reason:       ImagePullBackOff
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-hsc6d (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
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
Events:
  Type     Reason     Age                  From               Message
  ----     ------     ----                 ----               -------
  Normal   Scheduled  3m1s                 default-scheduler  Successfully assigned default/kubernetes-bootcamp-559647b9d-47stc to minikube
  Normal   Pulling    98s (x4 over 3m)     kubelet, minikube  Pulling image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v10"
  Warning  Failed     98s (x4 over 2m57s)  kubelet, minikube  Failed to pull image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v10": rpc error: code = Unknown desc = Error response from daemon: manifest for registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v10 not found
  Warning  Failed     98s (x4 over 2m57s)  kubelet, minikube  Error: ErrImagePull
  Normal   BackOff    71s (x6 over 2m57s)  kubelet, minikube  Back-off pulling image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v10"
  Warning  Failed     60s (x7 over 2m57s)  kubelet, minikube  Error: ImagePullBackOff
nothing@pc:~$
~~~

再次回退后，又回到错误的版本


~~~
nothing@pc:~$ kubectl rollout undo deployments/kubernetes-bootcamp
deployment.extensions/kubernetes-bootcamp rolled back
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-57cb7dcc4d-8nb6g   1/1     Running   0          26m
kubernetes-bootcamp-57cb7dcc4d-bm66b   1/1     Running   0          26m
kubernetes-bootcamp-57cb7dcc4d-ghk7r   1/1     Running   0          2s
kubernetes-bootcamp-57cb7dcc4d-kggh6   1/1     Running   0          26m
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           43m
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v3| 
nothing@pc:~$
~~~

再次回退后，又到了之前版本

可见默认的回退操作就是在最后两个版本上进行切换 

不过，这个行为可以被 **`--to-revision`** 调整

~~~
nothing@pc:~$ kubectl rollout undo deployments/kubernetes-bootcamp  --to-revision=1
deployment.extensions/kubernetes-bootcamp rolled back
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v1| 
nothing@pc:~$ kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           47m
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-57cb7dcc4d-8nb6g   1/1     Terminating   0          31m
kubernetes-bootcamp-57cb7dcc4d-bm66b   1/1     Terminating   0          31m
kubernetes-bootcamp-57cb7dcc4d-ghk7r   0/1     Terminating   0          4m28s
kubernetes-bootcamp-57cb7dcc4d-kggh6   1/1     Terminating   0          31m
kubernetes-bootcamp-5b7588689d-2zw9m   1/1     Running       0          29s
kubernetes-bootcamp-5b7588689d-9kfdb   1/1     Running       0          32s
kubernetes-bootcamp-5b7588689d-gtbbl   1/1     Running       0          32s
kubernetes-bootcamp-5b7588689d-gzlb8   1/1     Running       0          29s
nothing@pc:~$ 
~~~


~~~
nothing@pc:~$ kubectl rollout undo deployments/kubernetes-bootcamp  --to-revision=2
deployment.extensions/kubernetes-bootcamp rolled back
nothing@pc:~$ curl http://192.168.99.100:32306
Hello World!  |v2| 
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-58b56b5ff9-5jp5p   1/1     Running       0          8s
kubernetes-bootcamp-58b56b5ff9-cdpvw   1/1     Running       0          6s
kubernetes-bootcamp-58b56b5ff9-gn6kn   1/1     Running       0          8s
kubernetes-bootcamp-58b56b5ff9-rwvk5   1/1     Running       0          7s
kubernetes-bootcamp-5b7588689d-2zw9m   1/1     Terminating   0          63s
kubernetes-bootcamp-5b7588689d-9kfdb   1/1     Terminating   0          66s
kubernetes-bootcamp-5b7588689d-gtbbl   1/1     Terminating   0          66s
kubernetes-bootcamp-5b7588689d-gzlb8   1/1     Terminating   0          63s
nothing@pc:~$
~~~

并且因为有 service 作用中间层，升级过程对于前端应用来讲是基本无感的


## 查看升级记录

~~~
nothing@pc:~$ kubectl describe deployment/kubernetes-bootcamp
Name:                   kubernetes-bootcamp
Namespace:              default
CreationTimestamp:      Wed, 14 Aug 2019 17:17:19 +0800
Labels:                 run=kubernetes-bootcamp
Annotations:            deployment.kubernetes.io/revision: 10
Selector:               run=kubernetes-bootcamp
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  run=kubernetes-bootcamp
  Containers:
   kubernetes-bootcamp:
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
NewReplicaSet:   kubernetes-bootcamp-5b7588689d (4/4 replicas created)
Events:
  Type    Reason             Age                 From                   Message
  ----    ------             ----                ----                   -------
  Normal  ScalingReplicaSet  59m                 deployment-controller  Scaled up replica set kubernetes-bootcamp-5b7588689d to 4
  Normal  ScalingReplicaSet  46m                 deployment-controller  Scaled up replica set kubernetes-bootcamp-58b56b5ff9 to 1
  Normal  ScalingReplicaSet  46m                 deployment-controller  Scaled down replica set kubernetes-bootcamp-5b7588689d to 3
  Normal  ScalingReplicaSet  46m                 deployment-controller  Scaled up replica set kubernetes-bootcamp-58b56b5ff9 to 2
  Normal  ScalingReplicaSet  46m                 deployment-controller  Scaled down replica set kubernetes-bootcamp-5b7588689d to 2
  Normal  ScalingReplicaSet  46m                 deployment-controller  Scaled up replica set kubernetes-bootcamp-58b56b5ff9 to 3
  Normal  ScalingReplicaSet  46m                 deployment-controller  Scaled down replica set kubernetes-bootcamp-5b7588689d to 1
  Normal  ScalingReplicaSet  46m                 deployment-controller  Scaled up replica set kubernetes-bootcamp-58b56b5ff9 to 4
  Normal  ScalingReplicaSet  46m                 deployment-controller  Scaled down replica set kubernetes-bootcamp-5b7588689d to 0
  Normal  ScalingReplicaSet  42m                 deployment-controller  Scaled up replica set kubernetes-bootcamp-57cb7dcc4d to 1
  Normal  ScalingReplicaSet  24m                 deployment-controller  Scaled up replica set kubernetes-bootcamp-559647b9d to 1
  Normal  ScalingReplicaSet  15m                 deployment-controller  Scaled down replica set kubernetes-bootcamp-559647b9d to 0
  Normal  ScalingReplicaSet  12m (x2 over 24m)   deployment-controller  Scaled down replica set kubernetes-bootcamp-57cb7dcc4d to 3
  Normal  ScalingReplicaSet  11m (x22 over 42m)  deployment-controller  (combined from similar events): Scaled up replica set kubernetes-bootcamp-58b56b5ff9 to 1
nothing@pc:~$ 
~~~

通过  Event 栏可以看到相应事件

---

# 总结

主要是 kubectl 命令的使用

~~~
kubectl get deployment
kubectl run kubernetes-bootcamp --image=registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1 --port=8080  --replicas=4
kubectl get pods
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port=8080
minikube service kubernetes-bootcamp --url
curl http://192.168.99.100:32306
kubectl describe pods
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v2 
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v3 
kubectl describe service/kubernetes-bootcamp
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{\{(index .spec.ports 0).nodePort}\}')
echo NODE_PORT=$NODE_PORT
curl $(minikube ip):$NODE_PORT
kubectl rollout status deployments/kubernetes-bootcamp
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v10 
kubectl rollout undo deployments/kubernetes-bootcamp
kubectl describe pods/kubernetes-bootcamp-559647b9d-47stc
kubectl rollout undo deployments/kubernetes-bootcamp  --to-revision=1
kubectl rollout undo deployments/kubernetes-bootcamp  --to-revision=2
kubectl describe deployment/kubernetes-bootcamp
~~~


* TOC
{:toc}

---


[scale_interactive]:https://kubernetes.io/docs/tutorials/kubernetes-basics/scale/scale-interactive/
[kubernetes]:https://kubernetes.io/


