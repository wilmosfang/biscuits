---
layout: post
title: "Create a ConfigMap"
date: 2019-08-14 16:21:35
image: '/assets/img/'
description: '创建 ConfigMap'
main-class:  'kubernetes'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
 - kubernetes
twitter_text:  'simple way to create a configmap'
introduction: 'simple way to create a configmap'
---



# 前言


**[k8s][kubernetes]** 中 **ConfigMaps** 可以让应用与配置解偶，从而让应用具备更好的移植性

这篇讲如何在 **[k8s][kubernetes]** 中创建 **ConfigMaps** 

>ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized applications portable

这里讲讲 **ConfigMaps** 的几种常见创建方法 

参考 **[Configure a Pod to Use a ConfigMap][configure_pod_configmap]** 


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

## 通过 kubectl create configmap 来创建 ConfigMap

可以使用 **`kubectl create configmap`** 从以下三种形式

* 目录
* 文件
* 字符

创建 ConfigMap

**`kubectl create configmap <map-name> <data-source>`**

## 准备数据

~~~
nothing@pc:~$ mkdir -p configure-pod-container/configmap/
nothing@pc:~$ wget https://kubernetes.io/examples/configmap/game.properties -O configure-pod-container/configmap/game.properties
--2019-08-15 01:02:37--  https://kubernetes.io/examples/configmap/game.properties
正在解析主机 kubernetes.io (kubernetes.io)... 45.54.44.102
正在连接 kubernetes.io (kubernetes.io)|45.54.44.102|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 157 [application/octet-stream]
正在保存至: “configure-pod-container/configmap/game.properties”

configure-pod-conta 100%[===================>]     157  --.-KB/s    用时 0s    

2019-08-15 01:02:38 (2.27 MB/s) - 已保存 “configure-pod-container/configmap/game.properties” [157/157])

nothing@pc:~$ wget https://kubernetes.io/examples/configmap/ui.properties -O configure-pod-container/configmap/ui.properties
--2019-08-15 01:02:46--  https://kubernetes.io/examples/configmap/ui.properties
正在解析主机 kubernetes.io (kubernetes.io)... 45.54.44.102
正在连接 kubernetes.io (kubernetes.io)|45.54.44.102|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 83 [application/octet-stream]
正在保存至: “configure-pod-container/configmap/ui.properties”

configure-pod-conta 100%[===================>]      83  --.-KB/s    用时 0s    

2019-08-15 01:02:46 (1.80 MB/s) - 已保存 “configure-pod-container/configmap/ui.properties” [83/83])

nothing@pc:~$ ll configure-pod-container/configmap
总用量 16
drwxrwxr-x 2 nothing nothing 4096 Ago 15 01:02 ./
drwxrwxr-x 3 nothing nothing 4096 Ago 15 01:02 ../
-rw-rw-r-- 1 nothing nothing  157 Ago 15 01:02 game.properties
-rw-rw-r-- 1 nothing nothing   83 Ago 15 01:02 ui.properties
nothing@pc:~$
nothing@pc:~$ cat configure-pod-container/configmap/game.properties 
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30nothing@pc:~$ 
nothing@pc:~$ cat configure-pod-container/configmap/ui.properties 
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice
nothing@pc:~$ 
~~~

## 通过目录创建 ConfigMap

~~~
nothing@pc:~$ kubectl create configmap game-config --from-file=configure-pod-container/configmap/
configmap/game-config created
nothing@pc:~$ 
~~~

## 查看 ConfigMap 内容

~~~
nothing@pc:~$ kubectl get configmap game-config
NAME          DATA   AGE
game-config   2      19s
nothing@pc:~$ kubectl describe configmap game-config
Name:         game-config
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
ui.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

Events:  <none>
nothing@pc:~$ 
~~~

## 使用 yaml 的格式查看内容


~~~
nothing@pc:~$ kubectl get configmaps game-config -o yaml
apiVersion: v1
data:
  game.properties: |-
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
  ui.properties: |
    color.good=purple
    color.bad=yellow
    allow.textmode=true
    how.nice.to.look=fairlyNice
kind: ConfigMap
metadata:
  creationTimestamp: "2019-08-14T17:06:18Z"
  name: game-config
  namespace: default
  resourceVersion: "90053"
  selfLink: /api/v1/namespaces/default/configmaps/game-config
  uid: 00d53a4e-c38a-42b6-8762-5b61848114e7
nothing@pc:~$ 
~~~

## 从文件创建 ConfigMaps

~~~
nothing@pc:~$ kubectl create configmap game-config-2 --from-file=configure-pod-container/configmap/game.properties
configmap/game-config-2 created
nothing@pc:~$ 
~~~

查看内容 

~~~
nothing@pc:~$ kubectl describe configmap game-config-2
Name:         game-config-2
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
Events:  <none>
nothing@pc:~$ 
~~~


## 从多个文件创建 ConfigMaps


~~~
nothing@pc:~$ kubectl create configmap game-config-22 --from-file=configure-pod-container/configmap/game.properties --from-file=configure-pod-container/configmap/ui.properties
configmap/game-config-22 created
nothing@pc:~$ 
~~~

这种形式等价于指向目录

~~~
nothing@pc:~$ kubectl describe configmap game-config-22
Name:         game-config-22
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game.properties:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
ui.properties:
----
color.good=purple
color.bad=yellow
allow.textmode=true
how.nice.to.look=fairlyNice

Events:  <none>
nothing@pc:~$ 
~~~

## env-file 的方式构建 ConfigMaps

下载文件

~~~
nothing@pc:~$ wget https://kubernetes.io/examples/configmap/game-env-file.properties -O configure-pod-container/configmap/game-env-file.properties
--2019-08-15 01:15:02--  https://kubernetes.io/examples/configmap/game-env-file.properties
正在解析主机 kubernetes.io (kubernetes.io)... 45.54.44.102
正在连接 kubernetes.io (kubernetes.io)|45.54.44.102|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 94 [application/octet-stream]
正在保存至: “configure-pod-container/configmap/game-env-file.properties”

configure-pod-conta 100%[===================>]      94  --.-KB/s    用时 0s    

2019-08-15 01:15:03 (1.69 MB/s) - 已保存 “configure-pod-container/configmap/game-env-file.properties” [94/94])

nothing@pc:~$ cat configure-pod-container/configmap/game-env-file.properties
enemies=aliens
lives=3
allowed="true"

# This comment and the empty line above it are ignored
nothing@pc:~$ 
~~~



~~~
nothing@pc:~$ kubectl create configmap game-config-env-file  --from-env-file=configure-pod-container/configmap/game-env-file.properties
configmap/game-config-env-file created
nothing@pc:~$ 
nothing@pc:~$ 
nothing@pc:~$ kubectl get configmap game-config-env-file -o yaml
apiVersion: v1
data:
  allowed: '"true"'
  enemies: aliens
  lives: "3"
kind: ConfigMap
metadata:
  creationTimestamp: "2019-08-14T17:18:22Z"
  name: game-config-env-file
  namespace: default
  resourceVersion: "90933"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-env-file
  uid: 5b7bc83a-9d56-4f12-ab58-fdafc0e5fd00
nothing@pc:~$
~~~


## 多次导入 env-file 的方式构建 ConfigMaps

下载文件

~~~
nothing@pc:~$ wget https://kubernetes.io/examples/configmap/ui-env-file.properties -O configure-pod-container/configmap/ui-env-file.properties
--2019-08-15 01:20:48--  https://kubernetes.io/examples/configmap/ui-env-file.properties
正在解析主机 kubernetes.io (kubernetes.io)... 45.54.44.102
正在连接 kubernetes.io (kubernetes.io)|45.54.44.102|:443... 已连接。
已发出 HTTP 请求，正在等待回应... 200 OK
长度： 42 [application/octet-stream]
正在保存至: “configure-pod-container/configmap/ui-env-file.properties”

configure-pod-conta 100%[===================>]      42  --.-KB/s    用时 0s    

2019-08-15 01:20:48 (802 KB/s) - 已保存 “configure-pod-container/configmap/ui-env-file.properties” [42/42])

nothing@pc:~$ 
nothing@pc:~$ cat configure-pod-container/configmap/ui-env-file.properties
color=purple
textmode=true
how=fairlyNice
nothing@pc:~$ 
~~~

多次导入

~~~
nothing@pc:~$ kubectl create configmap config-multi-env-files --from-env-file=configure-pod-container/configmap/game-env-file.properties  --from-env-file=configure-pod-container/configmap/ui-env-file.properties
configmap/config-multi-env-files created
nothing@pc:~$ kubectl get configmap config-multi-env-files -o yaml
apiVersion: v1
data:
  color: purple
  how: fairlyNice
  textmode: "true"
kind: ConfigMap
metadata:
  creationTimestamp: "2019-08-14T17:22:30Z"
  name: config-multi-env-files
  namespace: default
  resourceVersion: "91235"
  selfLink: /api/v1/namespaces/default/configmaps/config-multi-env-files
  uid: 44666890-c362-4789-9be3-a3a15beafc1d
nothing@pc:~$ 
~~~

只有最后一个 env-file 的内容被导入


## 通过 key 来引用一个文件

**`kubectl create configmap game-config-3 --from-file=<my-key-name>=<path-to-file>`**

可以通过这种形式来让 key 引用 文件

~~~
nothing@pc:~$ kubectl create configmap game-config-3 --from-file=game-special-key=configure-pod-container/configmap/game.properties
configmap/game-config-3 created
nothing@pc:~$ kubectl get configmap game-config-3
NAME            DATA   AGE
game-config-3   1      12s
nothing@pc:~$ kubectl describe configmap game-config-3
Name:         game-config-3
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
game-special-key:
----
enemies=aliens
lives=3
enemies.cheat=true
enemies.cheat.level=noGoodRotten
secret.code.passphrase=UUDDLRLRBABAS
secret.code.allowed=true
secret.code.lives=30
Events:  <none>
nothing@pc:~$
nothing@pc:~$ kubectl get configmap game-config-3 -o yaml 
apiVersion: v1
data:
  game-special-key: |-
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
kind: ConfigMap
metadata:
  creationTimestamp: "2019-08-15T03:00:32Z"
  name: game-config-3
  namespace: default
  resourceVersion: "101687"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-3
  uid: 18416140-1ab3-43bc-a126-201481488d8f
nothing@pc:~$
~~~

## 通过字串创建 ConfigMaps

通过 **`--from-literal`** 来获取字符串中的 KV 对

~~~
nothing@pc:~$ kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
configmap/special-config created
nothing@pc:~$ kubectl get configmaps special-config -o yaml
apiVersion: v1
data:
  special.how: very
  special.type: charm
kind: ConfigMap
metadata:
  creationTimestamp: "2019-08-15T03:04:08Z"
  name: special-config
  namespace: default
  resourceVersion: "101948"
  selfLink: /api/v1/namespaces/default/configmaps/special-config
  uid: 592e8c1a-03be-410b-8988-06a39c1091b5
nothing@pc:~$ 
~~~

## 通过生成器来创建 ConfigMap

从 **`1.14`** 版本后 **kubectl** 就支持 **kustomization.yaml** 

可以通过生成器来创建 ConfigMap 对象

生成器必须在一个目录下的 **kustomization.yaml** 中先配置好

### 通过文件生成 

~~~
nothing@pc:~$ cat <<EOF >./kustomization.yaml
> configMapGenerator:
> - name: game-config-4
>   files:
>   - configure-pod-container/configmap/game.properties
> EOF
nothing@pc:~$ cat kustomization.yaml 
configMapGenerator:
- name: game-config-4
  files:
  - configure-pod-container/configmap/game.properties
nothing@pc:~$
nothing@pc:~$ kubectl apply -k . 
configmap/game-config-4-m9dm2f92bt created
nothing@pc:~$ kubectl get configmap
NAME                       DATA   AGE
config-multi-env-files     3      9h
game-config                2      10h
game-config-2              1      10h
game-config-22             2      10h
game-config-3              1      18m
game-config-4-m9dm2f92bt   1      20s
game-config-env-file       3      10h
special-config             2      14m
nothing@pc:~$ kubectl get configmap/game-config-4-m9dm2f92bt -o yaml 
apiVersion: v1
data:
  game.properties: |-
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"game.properties":"enemies=aliens\nlives=3\nenemies.cheat=true\nenemies.cheat.level=noGoodRotten\nsecret.code.passphrase=UUDDLRLRBABAS\nsecret.code.allowed=true\nsecret.code.lives=30"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"game-config-4-m9dm2f92bt","namespace":"default"}}
  creationTimestamp: "2019-08-15T03:18:31Z"
  name: game-config-4-m9dm2f92bt
  namespace: default
  resourceVersion: "102995"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-4-m9dm2f92bt
  uid: dff9dbaa-a6a6-4846-8055-bf0063e43ed3
nothing@pc:~$ 
~~~

>**Note:** 创建出来的 ConfigMap 有一个 hash 后缀，这是为了确保新的 ConfigMap 在内容变更后会是一个新的名字

### 用 key 来指向文件的方式生成

~~~
nothing@pc:~$ cat <<EOF >./kustomization.yaml
> configMapGenerator:
> - name: game-config-5
>   files:
>   - game-special-key=configure-pod-container/configmap/game.properties
> EOF
nothing@pc:~$ cat kustomization.yaml 
configMapGenerator:
- name: game-config-5
  files:
  - game-special-key=configure-pod-container/configmap/game.properties
nothing@pc:~$ kubectl apply -k . 
configmap/game-config-5-m67dt67794 created
nothing@pc:~$ 
nothing@pc:~$ kubectl get configmap/game-config-5-m67dt67794 -o yaml 
apiVersion: v1
data:
  game-special-key: |-
    enemies=aliens
    lives=3
    enemies.cheat=true
    enemies.cheat.level=noGoodRotten
    secret.code.passphrase=UUDDLRLRBABAS
    secret.code.allowed=true
    secret.code.lives=30
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"game-special-key":"enemies=aliens\nlives=3\nenemies.cheat=true\nenemies.cheat.level=noGoodRotten\nsecret.code.passphrase=UUDDLRLRBABAS\nsecret.code.allowed=true\nsecret.code.lives=30"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"game-config-5-m67dt67794","namespace":"default"}}
  creationTimestamp: "2019-08-15T03:28:36Z"
  name: game-config-5-m67dt67794
  namespace: default
  resourceVersion: "103731"
  selfLink: /api/v1/namespaces/default/configmaps/game-config-5-m67dt67794
  uid: c9a5dd55-ef98-4fea-b8e1-834e33c5ae11
nothing@pc:~$ 
~~~

### 通过字符创建


~~~
nothing@pc:~$ cat <<EOF >./kustomization.yaml
> configMapGenerator:
> - name: special-config-2
>   literals:
>   - special.how=very
>   - special.type=charm
> EOF
nothing@pc:~$ kubectl apply -k . 
configmap/special-config-2-c92b5mmcf2 created
nothing@pc:~$ kubectl get configmap/special-config-2-c92b5mmcf2 -o yaml 
apiVersion: v1
data:
  special.how: very
  special.type: charm
kind: ConfigMap
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","data":{"special.how":"very","special.type":"charm"},"kind":"ConfigMap","metadata":{"annotations":{},"name":"special-config-2-c92b5mmcf2","namespace":"default"}}
  creationTimestamp: "2019-08-15T03:34:42Z"
  name: special-config-2-c92b5mmcf2
  namespace: default
  resourceVersion: "104174"
  selfLink: /api/v1/namespaces/default/configmaps/special-config-2-c92b5mmcf2
  uid: aab335a9-afc8-48a6-9c40-8f1855189464
nothing@pc:~$ 
~~~


以上就是通过各种方式创建 ConfigMap 对象的方式

---

# 总结

主要是 kubectl 命令的使用

~~~
mkdir -p configure-pod-container/configmap/
wget https://kubernetes.io/examples/configmap/game.properties -O configure-pod-container/configmap/game.properties
wget https://kubernetes.io/examples/configmap/ui.properties -O configure-pod-container/configmap/ui.properties
kubectl create configmap game-config --from-file=configure-pod-container/configmap/
kubectl get configmap game-config
kubectl describe configmap game-config
kubectl get configmaps game-config -o yaml
kubectl create configmap game-config-2 --from-file=configure-pod-container/configmap/game.properties
kubectl create configmap game-config-22 --from-file=configure-pod-container/configmap/game.properties --from-file=configure-pod-container/configmap/ui.properties
wget https://kubernetes.io/examples/configmap/game-env-file.properties -O configure-pod-container/configmap/game-env-file.properties
cat configure-pod-container/configmap/game-env-file.properties
kubectl create configmap game-config-env-file  --from-env-file=configure-pod-container/configmap/game-env-file.properties
wget https://kubernetes.io/examples/configmap/ui-env-file.properties -O configure-pod-container/configmap/ui-env-file.properties
cat configure-pod-container/configmap/ui-env-file.properties
kubectl create configmap config-multi-env-files --from-env-file=configure-pod-container/configmap/game-env-file.properties  --from-env-file=configure-pod-container/configmap/ui-env-file.properties
kubectl get configmap config-multi-env-files -o yaml
kubectl create configmap game-config-3 --from-file=game-special-key=configure-pod-container/configmap/game.properties
kubectl create configmap special-config --from-literal=special.how=very --from-literal=special.type=charm
kubectl get configmaps special-config -o yaml
kubectl apply -k . 
kubectl get configmap
kubectl get configmap/game-config-4-m9dm2f92bt -o yaml 
~~~


* TOC
{:toc}

---


[configure_pod_configmap]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/
[kubernetes]:https://kubernetes.io/


