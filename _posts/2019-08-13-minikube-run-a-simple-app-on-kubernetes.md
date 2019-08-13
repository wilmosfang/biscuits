---
layout: post
title: "Run a Simple App on Kubernetes"
date: 2019-08-13 06:54:32
image: '/assets/img/'
description: 'Kubernetes 中运行一个小程序'
main-class:  'tools'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
- tools
twitter_text:  'run a simple app on kubernetes by minikube '
introduction: 'run a simple app on kubernetes by minikube '
---


# 前言

**[Minikube][minikube]** 是一个用来本地安装 **[Kubernetes][kubernetes]** 的简化工具 

>Minikube is a tool that makes it easy to run Kubernetes locally. Minikube runs a single-node Kubernetes cluster inside a Virtual Machine (VM) on your laptop for users looking to try out Kubernetes or develop with it day-to-day

当前 **[Kubernetes][kubernetes]** 非常火热，几乎成了新一代运维体系中的基础架构

为了对其特性进行了解

这里对 **[Minikube][minikube]** 的基础使用，进行一个简单的演示

通过 **[Minikube][minikube]** 来构建一个 web 小应用 

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


## 登录到 k8s 虚拟机中

~~~
nothing@pc:~$ minikube ssh
                         _             _            
            _         _ ( )           ( )           
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __  
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

$ 
$ 
~~~

## 构建 Docker file

~~~
$ mkdir minikube
$ ls
minikube
$ cd minikube/
$ ls
$ pwd
/home/docker/minikube
$ vi server.js 
$ cat server.js 
var http = require('http');

var handleRequest = function(request, response) {
  console.log('Received request for URL: ' + request.url);
  response.writeHead(200);
  response.end('Hello World!');
};
var www = http.createServer(handleRequest);
www.listen(8080);
$ vi Dockerfile 
$ cat Dockerfile 
FROM node:6.14.2
EXPOSE 8080
COPY server.js .
CMD node server.js
$ ls
Dockerfile  server.js
$ 
~~~

## 构建容器

~~~
$ docker build . 
Sending build context to Docker daemon  3.072kB
Step 1/4 : FROM node:6.14.2
6.14.2: Pulling from library/node
3d77ce4481b1: Pull complete 
7d2f32934963: Pull complete 
0c5cf711b890: Pull complete 
9593dc852d6b: Pull complete 
4e3b8a1eb914: Pull complete 
ddcf13cc1951: Pull complete 
2e460d114172: Pull complete 
d94b1226fbf2: Pull complete 
Digest: sha256:62b9d88be259a344eb0b4e0dd1b12347acfe41c1bb0f84c3980262f8032acc5a
Status: Downloaded newer image for node:6.14.2
 ---> 00165cd5d0c0
Step 2/4 : EXPOSE 8080
 ---> Running in 41aecd431b0a
Removing intermediate container 41aecd431b0a
 ---> 19768e059018
Step 3/4 : COPY server.js .
 ---> 51c77a73842f
Step 4/4 : CMD node server.js
 ---> Running in 7ab0f69969b8
Removing intermediate container 7ab0f69969b8
 ---> 20e88fca0cb8
Successfully built 20e88fca0cb8
$ docker images
REPOSITORY                                                                        TAG                 IMAGE ID            CREATED             SIZE
<none>                                                                            <none>              20e88fca0cb8        9 seconds ago       660MB
daocloud.io/library/nginx                                                         latest              98ebf73aba75        3 weeks ago         109MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy                    v1.15.0             d235b23c3570        7 weeks ago         82.4MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver                v1.15.0             201c7a840312        7 weeks ago         207MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler                v1.15.0             2d3813851e87        7 weeks ago         81.1MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager       v1.15.0             8328bb49b652        7 weeks ago         159MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-addon-manager            v9.0                119701e77cbc        6 months ago        83.1MB
registry.cn-hangzhou.aliyuncs.com/google_containers/coredns                       1.3.1               eb516548c180        7 months ago        40.3MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64    v1.10.1             f9aed6605b81        7 months ago        122MB
registry.cn-hangzhou.aliyuncs.com/google_containers/etcd                          3.3.10              2c4adeb21b4f        8 months ago        258MB
registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-sidecar-amd64         1.14.13             4b2e93f0133d        10 months ago       42.9MB
registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-kube-dns-amd64        1.14.13             55a3c5209c5e        10 months ago       51.2MB
registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-dnsmasq-nanny-amd64   1.14.13             6dc8ef8287d3        10 months ago       41.4MB
node                                                                              6.14.2              00165cd5d0c0        14 months ago       660MB
registry.cn-hangzhou.aliyuncs.com/google_containers/pause                         3.1                 da86e6ba6ca1        20 months ago       742kB
registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner           v1.8.1              4689081edb10        21 months ago       80.8MB
$ 
$ docker tag 20e88fca0cb8 local/hello-node
$ docker images
REPOSITORY                                                                        TAG                 IMAGE ID            CREATED             SIZE
local/hello-node                                                                  latest              20e88fca0cb8        2 minutes ago       660MB
daocloud.io/library/nginx                                                         latest              98ebf73aba75        3 weeks ago         109MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy                    v1.15.0             d235b23c3570        7 weeks ago         82.4MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver                v1.15.0             201c7a840312        7 weeks ago         207MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager       v1.15.0             8328bb49b652        7 weeks ago         159MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler                v1.15.0             2d3813851e87        7 weeks ago         81.1MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-addon-manager            v9.0                119701e77cbc        6 months ago        83.1MB
registry.cn-hangzhou.aliyuncs.com/google_containers/coredns                       1.3.1               eb516548c180        7 months ago        40.3MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kubernetes-dashboard-amd64    v1.10.1             f9aed6605b81        7 months ago        122MB
registry.cn-hangzhou.aliyuncs.com/google_containers/etcd                          3.3.10              2c4adeb21b4f        8 months ago        258MB
registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-sidecar-amd64         1.14.13             4b2e93f0133d        10 months ago       42.9MB
registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-kube-dns-amd64        1.14.13             55a3c5209c5e        10 months ago       51.2MB
registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-dns-dnsmasq-nanny-amd64   1.14.13             6dc8ef8287d3        10 months ago       41.4MB
node                                                                              6.14.2              00165cd5d0c0        14 months ago       660MB
registry.cn-hangzhou.aliyuncs.com/google_containers/pause                         3.1                 da86e6ba6ca1        20 months ago       742kB
registry.cn-hangzhou.aliyuncs.com/google_containers/storage-provisioner           v1.8.1              4689081edb10        21 months ago       80.8MB
$ 
~~~

镜像已经构建好了，同时打上了本地的标


## 推送镜像到仓库

我这里推到了阿里云的仓库中


~~~
$ sudo docker login --username=wmscpt@sina.com registry.cn-hangzhou.aliyuncs.com
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
$ 
$ sudo docker tag 20e88fca0cb8 registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1.0
$ sudo docker push registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1.0
The push refers to repository [registry.cn-hangzhou.aliyuncs.com/wilmos/k8s]
0b3da8514827: Pushed 
aeaa1edefd60: Pushed 
6e650662f0e3: Pushed 
8c825a971eaf: Pushed 
bf769027dbbd: Pushed 
f3693db46abb: Pushed 
bb6d734b467e: Pushed 
5f349fdc9028: Pushed 
2c833f307fd8: Pushed 
v1.0: digest: sha256:63442fc4c863bfcddbc4e5360ddae91c561f66cf1864b5a14ba0bc29a1412727 size: 2214
$ 
~~~

>**Note:**  推送之前是有一个配置过程的，这个过程在阿里云控制台中完成

![k8s](/assets/img/k8s/k8s04.png)


推送完成后，仓库中会有一个镜像

![k8s](/assets/img/k8s/k8s05.png)

## 创建一个 Deployment

~~~
nothing@pc:~$ kubectl get deployment
No resources found.
nothing@pc:~$ kubectl create deployment hello-node --image=registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1.0
deployment.apps/hello-node created
nothing@pc:~$ kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   1/1     1            1           8s
nothing@pc:~$ 
~~~

## 查看 pod

~~~
nothing@pc:~$ kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
hello-node-6c6c556c64-npnmr   1/1     Running   0          15s
nothing@pc:~$
~~~

## 查看集群事件

~~~
nothing@pc:~$ kubectl get events
LAST SEEN   TYPE      REASON              OBJECT                             MESSAGE
20m         Normal    Scheduled           pod/hello-node-55f9c56776-6d9b8    Successfully assigned default/hello-node-55f9c56776-6d9b8 to minikube
17m         Normal    Pulling             pod/hello-node-55f9c56776-6d9b8    Pulling image "local/hello-node"
17m         Warning   Failed              pod/hello-node-55f9c56776-6d9b8    Failed to pull image "local/hello-node": rpc error: code = Unknown desc = Error response from daemon: pull access denied for local/hello-node, repository does not exist or may require 'docker login'
17m         Warning   Failed              pod/hello-node-55f9c56776-6d9b8    Error: ErrImagePull
16m         Normal    BackOff             pod/hello-node-55f9c56776-6d9b8    Back-off pulling image "local/hello-node"
5m11s       Warning   Failed              pod/hello-node-55f9c56776-6d9b8    Error: ImagePullBackOff
20m         Normal    SuccessfulCreate    replicaset/hello-node-55f9c56776   Created pod: hello-node-55f9c56776-6d9b8
25s         Normal    Scheduled           pod/hello-node-6c6c556c64-npnmr    Successfully assigned default/hello-node-6c6c556c64-npnmr to minikube
24s         Normal    Pulled              pod/hello-node-6c6c556c64-npnmr    Container image "registry.cn-hangzhou.aliyuncs.com/wilmos/k8s:v1.0" already present on machine
24s         Normal    Created             pod/hello-node-6c6c556c64-npnmr    Created container k8s
24s         Normal    Started             pod/hello-node-6c6c556c64-npnmr    Started container k8s
25s         Normal    SuccessfulCreate    replicaset/hello-node-6c6c556c64   Created pod: hello-node-6c6c556c64-npnmr
20m         Normal    ScalingReplicaSet   deployment/hello-node              Scaled up replica set hello-node-55f9c56776 to 1
25s         Normal    ScalingReplicaSet   deployment/hello-node              Scaled up replica set hello-node-6c6c556c64 to 1
nothing@pc:~$
~~~

## 查看 kubectl 的配置

~~~
nothing@pc:~$ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/nothing/.minikube/ca.crt
    server: https://192.168.99.101:8443
  name: minikube
contexts:
- context:
    cluster: minikube
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /home/nothing/.minikube/client.crt
    client-key: /home/nothing/.minikube/client.key
nothing@pc:~$ 
~~~

## 创建 Service

默认情况下 Pod 只能通过内部 IP 在 K8s 集群的内部访问

如果要使之能在外部访问，必须通过服务的方式将 Pod 暴露出来 

>By default, the Pod is only accessible by its internal IP address within the Kubernetes cluster. To make the hello-node Container accessible from outside the Kubernetes virtual network, you have to expose the Pod as a Kubernetes Service

~~~
nothing@pc:~$ kubectl expose deployment hello-node --type=LoadBalancer --port=8080
service/hello-node exposed
nothing@pc:~$
~~~


## 查看服务 

~~~
nothing@pc:~$  kubectl get services
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
hello-node   LoadBalancer   10.103.66.139   <pending>     8080:30655/TCP   41m
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          4h42m
nothing@pc:~$ 
nothing@pc:~$ minikube service hello-node --url
http://192.168.99.101:30655
nothing@pc:~$
~~~

## 进行访问

直接访问 **`http://192.168.99.101:30655`**

或者 

~~~
nothing@pc:~$ minikube service hello-node
* Opening kubernetes service default/hello-node in default browser...
nothing@pc:~$ 正在现有的浏览器会话中打开。
[26132:26160:0813/153439.321892:ERROR:browser_process_sub_thread.cc(221)] Waited 4 ms for network service
...
...
...
~~~

会弹出一个浏览器，展示 hello-node 中的内容

![k8s](/assets/img/k8s/k8s06.png)



---

# 总结

使用 **minikube** 对 **k8s** 集群进行创建和销毁

使用 **kubectl** 对 **k8s** 集群对象或资源进行管理



* TOC
{:toc}

---

[minikube]:https://kubernetes.io/docs/setup/learning-environment/minikube/
[kubernetes]:https://kubernetes.io/
[hello_minikube]:https://kubernetes.io/docs/tutorials/hello-minikube/


