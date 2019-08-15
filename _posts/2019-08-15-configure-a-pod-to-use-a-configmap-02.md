---
layout: post
title: "Configure a Pod to Use a ConfigMap"
date: 2019-08-15 05:56:36
image: '/assets/img/'
description: '在 Pod 中应用 ConfigMap'
main-class:  'kubernetes'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
 - kubernetes
twitter_text:  'simple way to use a configmap'
introduction: 'simple way to use a configmap'
---


# 前言


**[k8s][kubernetes]** 中 **ConfigMaps** 可以让应用与配置解偶，从而让应用具备更好的移植性

这篇讲如何在 **[k8s][kubernetes]** 中应用 **ConfigMaps** 

>ConfigMaps allow you to decouple configuration artifacts from image content to keep containerized applications portable

这里讲讲 **ConfigMaps** 的几种常见应用方法 

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

## 定义容器环境变量的方式来使用 ConfigMap

首先创建一个 ConfigMap，里面包含一个 KV 对

~~~
nothing@pc:~$ kubectl create configmap special-config --from-literal=special.how=very
configmap/special-config created
nothing@pc:~$ kubectl get configmap/special-config -o yaml 
apiVersion: v1
data:
  special.how: very
kind: ConfigMap
metadata:
  creationTimestamp: "2019-08-15T06:06:34Z"
  name: special-config
  namespace: default
  resourceVersion: "115219"
  selfLink: /api/v1/namespaces/default/configmaps/special-config
  uid: 8ecf8194-68d3-49d8-9538-28f97a9d32c8
nothing@pc:~$ 
~~~

再创建一个 pod 的配置文件

~~~
nothing@pc:/tmp$ vim env-variable.yaml
nothing@pc:/tmp$ cat env-variable.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: daocloud.io/library/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        # Define the environment variable
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config
              # Specify the key associated with the value
              key: special.how
  restartPolicy: Never
nothing@pc:/tmp$ 
~~~

创建 pod

~~~
nothing@pc:/tmp$ kubectl create -f env-variable.yaml 
pod/dapi-test-pod created
nothing@pc:/tmp$ kubectl get pods
NAME                                   READY   STATUS      RESTARTS   AGE
dapi-test-pod                          0/1     Completed   0          12s
kubernetes-bootcamp-5b7588689d-5txtt   1/1     Running     1          20h
kubernetes-bootcamp-5b7588689d-9jxzw   1/1     Running     1          20h
kubernetes-bootcamp-5b7588689d-jtqh9   1/1     Running     1          20h
kubernetes-bootcamp-5b7588689d-xhzps   1/1     Running     1          20h
nothing@pc:/tmp$ 
~~~

查看输出

~~~
nothing@pc:/tmp$ kubectl logs dapi-test-pod
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=dapi-test-pod
SHLVL=1
HOME=/root
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_BOOTCAMP_SERVICE_HOST=10.101.119.17
KUBERNETES_BOOTCAMP_PORT_8080_TCP_ADDR=10.101.119.17
KUBERNETES_BOOTCAMP_PORT_8080_TCP_PORT=8080
SPECIAL_LEVEL_KEY=very
KUBERNETES_BOOTCAMP_PORT_8080_TCP_PROTO=tcp
KUBERNETES_BOOTCAMP_SERVICE_PORT=8080
KUBERNETES_BOOTCAMP_PORT=tcp://10.101.119.17:8080
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
KUBERNETES_BOOTCAMP_PORT_8080_TCP=tcp://10.101.119.17:8080
nothing@pc:/tmp$ 
~~~

可以看到 ConfigMap 中的信息传递到了环境变量中

~~~
nothing@pc:/tmp$ kubectl logs dapi-test-pod | grep -i SPECIAL_LEVEL_KEY
SPECIAL_LEVEL_KEY=very
nothing@pc:/tmp$ 
~~~

## 从多个 ConfigMaps 中获取信息到容器的环境变量

首先创建两个 ConfigMap 的配置文件

~~~
nothing@pc:/tmp$ vim configmap.yaml 
nothing@pc:/tmp$ cat configmap.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  special.how: very
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: env-config
  namespace: default
data:
  log_level: INFO
nothing@pc:/tmp$
~~~

创建两个 ConfigMap 


~~~
nothing@pc:/tmp$ kubectl create -f configmap.yaml 
configmap/special-config created
configmap/env-config created
nothing@pc:/tmp$ 
nothing@pc:/tmp$ kubectl get configmap 
NAME             DATA   AGE
env-config       1      6s
special-config   1      6s
nothing@pc:/tmp$ kubectl get configmap/env-config -o yaml 
apiVersion: v1
data:
  log_level: INFO
kind: ConfigMap
metadata:
  creationTimestamp: "2019-08-15T06:34:57Z"
  name: env-config
  namespace: default
  resourceVersion: "117353"
  selfLink: /api/v1/namespaces/default/configmaps/env-config
  uid: 413c1a54-5758-47a6-9995-8d1f06e7a5a8
nothing@pc:/tmp$ kubectl get configmap/special-config -o yaml 
apiVersion: v1
data:
  special.how: very
kind: ConfigMap
metadata:
  creationTimestamp: "2019-08-15T06:34:57Z"
  name: special-config
  namespace: default
  resourceVersion: "117352"
  selfLink: /api/v1/namespaces/default/configmaps/special-config
  uid: 6d857cfb-b8b8-44fe-b423-8e88610c649d
nothing@pc:/tmp$ 
~~~

创建 pod 并且引用 ConfigMap 到环境变量中来

~~~
nothing@pc:/tmp$ vim env-variable.yaml 
nothing@pc:/tmp$ cat env-variable.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: daocloud.io/library/busybox
      command: [ "/bin/sh", "-c", "env" ]
      env:
        # Define the environment variable
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              # The ConfigMap containing the value you want to assign to SPECIAL_LEVEL_KEY
              name: special-config
              # Specify the key associated with the value
              key: special.how
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: env-config
              key: log_level
  restartPolicy: Never
nothing@pc:/tmp$ kubectl create -f env-variable.yaml 
pod/dapi-test-pod created
nothing@pc:/tmp$ kubectl get pods
NAME            READY   STATUS      RESTARTS   AGE
dapi-test-pod   0/1     Completed   0          4s
nothing@pc:/tmp$ kubectl logs dapi-test-pod
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
LOG_LEVEL=INFO
HOSTNAME=dapi-test-pod
SHLVL=1
HOME=/root
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
SPECIAL_LEVEL_KEY=very
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
nothing@pc:/tmp$ kubectl logs dapi-test-pod | egrep -i '(LOG_LEVEL|SPECIAL_LEVEL_KEY)'
LOG_LEVEL=INFO
SPECIAL_LEVEL_KEY=very
nothing@pc:/tmp$ 
~~~

## 将 ConfigMap 中的所有 KV 都传递到容器中作为环境变量


首先创建 ConfigMap 

~~~
nothing@pc:/tmp$ vim config
config-err-M2qDyv  configmap.yaml     
nothing@pc:/tmp$ vim configmap.yaml 
nothing@pc:/tmp$ cat configmap.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
nothing@pc:/tmp$ kubectl create -f configmap.yaml 
configmap/special-config created
nothing@pc:/tmp$ kubectl get configmap
NAME             DATA   AGE
special-config   2      10s
nothing@pc:/tmp$ kubectl get configmap -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    SPECIAL_LEVEL: very
    SPECIAL_TYPE: charm
  kind: ConfigMap
  metadata:
    creationTimestamp: "2019-08-15T07:16:06Z"
    name: special-config
    namespace: default
    resourceVersion: "120375"
    selfLink: /api/v1/namespaces/default/configmaps/special-config
    uid: 94df9f32-7cd0-4fa9-aca6-3d57bcade149
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
nothing@pc:/tmp$ 
~~~




~~~
nothing@pc:/tmp$ vim env-variable.yaml 
nothing@pc:/tmp$ cat env-variable.yaml 
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: daocloud.io/library/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: special-config
  restartPolicy: Never
nothing@pc:/tmp$ 
nothing@pc:/tmp$ kubectl get configmap
NAME             DATA   AGE
special-config   2      72m
nothing@pc:/tmp$
nothing@pc:/tmp$ kubectl create -f env-variable.yaml 
pod/dapi-test-pod created
nothing@pc:/tmp$ kubectl get pods
NAME            READY   STATUS      RESTARTS   AGE
dapi-test-pod   0/1     Completed   0          13s
nothing@pc:/tmp$ kubectl logs dapi-test-pod 
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.96.0.1:443
HOSTNAME=dapi-test-pod
SHLVL=1
HOME=/root
SPECIAL_LEVEL=very
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_SERVICE_HOST=10.96.0.1
PWD=/
SPECIAL_TYPE=charm
nothing@pc:/tmp$ 
~~~

ConfigMap 中的 KV 成功传递到了容器的环境中去了

## 引用 ConfigMap 中的信息到 Pod 的命令行中

在 pod 的命令行中，可以通过 **`$(VAR_NAME)`** 的方式引用环境变量里 ConfigMap 中的值

~~~
nothing@pc:/tmp$ vim env-variable.yaml 
nothing@pc:/tmp$ cat env-variable.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
---
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
  namespace: default
spec:
  containers:
    - name: test-container
      image: daocloud.io/library/busybox
      command: [ "/bin/sh", "-c", "echo $(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)" ]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: SPECIAL_LEVEL
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: SPECIAL_TYPE
  restartPolicy: Never
nothing@pc:/tmp$ 
~~~

创建 configmap 和 pod 并且校验信息

~~~
nothing@pc:/tmp$ kubectl get configmap
No resources found.
nothing@pc:/tmp$ kubectl get pods
No resources found.
nothing@pc:/tmp$
nothing@pc:/tmp$ kubectl create -f env-variable.yaml 
configmap/special-config created
pod/dapi-test-pod created
nothing@pc:/tmp$ kubectl get pods
NAME            READY   STATUS              RESTARTS   AGE
dapi-test-pod   0/1     ContainerCreating   0          7s
nothing@pc:/tmp$ kubectl get configmap
NAME             DATA   AGE
special-config   2      12s
nothing@pc:/tmp$ kubectl get configmap -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    SPECIAL_LEVEL: very
    SPECIAL_TYPE: charm
  kind: ConfigMap
  metadata:
    creationTimestamp: "2019-08-15T08:45:17Z"
    name: special-config
    namespace: default
    resourceVersion: "126906"
    selfLink: /api/v1/namespaces/default/configmaps/special-config
    uid: 14ac9ea6-6c4a-4b85-a614-6e00e31a6d84
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
nothing@pc:/tmp$ kubectl logs dapi-test-pod
very charm
nothing@pc:/tmp$ 
~~~

可见，configmap 中的信息被 pod 在命令行中进行了引用 


## 用 ConfigMap 来填充一个 Volume



~~~
nothing@pc:/tmp$ kubectl get pods
No resources found.
nothing@pc:/tmp$ kubectl get configmap
No resources found.
nothing@pc:/tmp$
nothing@pc:/tmp$ cat env-variable.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
---
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
  namespace: default
spec:
  containers:
    - name: test-container
      image: daocloud.io/library/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config " ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
  restartPolicy: Never
nothing@pc:/tmp$ kubectl create -f env-variable.yaml 
configmap/special-config created
pod/dapi-test-pod created
nothing@pc:/tmp$ kubectl get configmap -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    SPECIAL_LEVEL: very
    SPECIAL_TYPE: charm
  kind: ConfigMap
  metadata:
    creationTimestamp: "2019-08-15T09:21:17Z"
    name: special-config
    namespace: default
    resourceVersion: "129549"
    selfLink: /api/v1/namespaces/default/configmaps/special-config
    uid: cc1d3357-d190-4e46-9fd2-94b5f6b3499f
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
nothing@pc:/tmp$ kubectl get pods
NAME            READY   STATUS      RESTARTS   AGE
dapi-test-pod   0/1     Completed   0          21s
nothing@pc:/tmp$ kubectl logs dapi-test-pod
SPECIAL_LEVEL
SPECIAL_TYPE
verycharmnothing@pc:/tmp$
~~~

>**Note:** 如果 **`/etc/config/`** 目录下面有文件，它们将被删除

## 将 ConfigMap 的数据添加到卷的指定路径中


~~~
nothing@pc:/tmp$ kubectl get configmap
No resources found.
nothing@pc:/tmp$ kubectl get pods
No resources found.
nothing@pc:/tmp$ vim env-variable.yaml 
nothing@pc:/tmp$ cat env-variable.yaml 
apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
---
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
  namespace: default
spec:
  containers:
    - name: test-container
      image: daocloud.io/library/busybox
      command: [ "/bin/sh", "-c", "cat /etc/config/keys" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        # Provide the name of the ConfigMap containing the files you want
        # to add to the container
        name: special-config
        items:
        - key: SPECIAL_LEVEL
          path: keys
  restartPolicy: Never
nothing@pc:/tmp$ kubectl create -f env-variable.yaml 
configmap/special-config created
pod/dapi-test-pod created
nothing@pc:/tmp$  kubectl get pods
NAME            READY   STATUS      RESTARTS   AGE
dapi-test-pod   0/1     Completed   0          15s
nothing@pc:/tmp$ kubectl get configmap -o yaml
apiVersion: v1
items:
- apiVersion: v1
  data:
    SPECIAL_LEVEL: very
    SPECIAL_TYPE: charm
  kind: ConfigMap
  metadata:
    creationTimestamp: "2019-08-15T09:34:34Z"
    name: special-config
    namespace: default
    resourceVersion: "130532"
    selfLink: /api/v1/namespaces/default/configmaps/special-config
    uid: 2ea9fd0a-5daa-49d3-9ed0-f03d7d861a01
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
nothing@pc:/tmp$ kubectl logs dapi-test-pod
verynothing@pc:/tmp$ 
nothing@pc:/tmp$ 
nothing@pc:/tmp$ 
~~~

当 ConfigMap 被充当一个卷时被更新，那么项目中的 KV 也会被更新

在周期性的检查中，K8s 会检查 ConfigMap 是否被刷新

一般而言，会有一个本地的 TTL 时间来获取最新的值

这个值一般是 kubelet 的同步周期，默认情况下会是 1 分钟 

和 ConfigMaps 的缓冲周期，默认情况下也是 1 分钟，的总和

所以会是 2 分钟


## ConfigMaps 和 Pods 的关系

ConfigMap 以 KV 对的方式来存储配置信息

这个信息可以被 Pods 来调用，或者 controllers 一类的系统组件使用

ConfigMap 类似于 Secrets ，但是提供的是明文字串信息而不是敏感信息

用户和系统一样也可以存储配置信息到 ConfigMap 中

>**Note:** ConfigMap 应该参考属性文件，而不是替换它们，ConfigMap 代表 /etc 下的目录和内容，如果通过 ConfigMap 创建了一个 K8s 卷，那么 ConfigMap 中的每一个 item 都代表卷中的一个独立文件

## 限制

* 被调用的 ConfigMap 要在 Pod 之前创建，除非此 ConfigMap 是被指定为可选的，如果指定的 ConfigMap 不存在，那么 Pod 就不会启动，同样，所引用的 key 不存在的情况下，Pod 也不会启动
* 如果使用 envFrom 的方式来定义环境变量，无效的 keys 会被忽略，pod 可以启动，但是无效的变量名会被记录到事件日志中
* ConfigMap 在相同的 namespace 中有效，只能被相同的 namespace 中的 pod 引用
* 在 API server 中找不到的 Pods， kubelet 是不支持它使用 ConfigMaps 的


以上就是常见使用 ConfigMap 对象的方式

---

# 总结

主要是 kubectl 命令的使用

~~~
kubectl create -f configmap.yaml 
kubectl create -f env-variable.yaml 
kubectl get configmap/env-config -o yaml 
kubectl get configmap/special-config -o yaml 
kubectl logs dapi-test-pod
kubectl logs dapi-test-pod | egrep -i '(LOG_LEVEL|SPECIAL_LEVEL_KEY)'
~~~


* TOC
{:toc}

---


[configure_pod_configmap]:https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data
[kubernetes]:https://kubernetes.io/


