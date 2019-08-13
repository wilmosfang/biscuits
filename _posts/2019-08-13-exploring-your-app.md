---
layout: post
title: "Exploring the App"
date: 2019-08-13 15:42:08
image: '/assets/img/'
description: '深入到应用容器中'
main-class:  'tools'
color: 
tags:
 - docker
 - kubernetes
 - kubectl
 - minikube
categories: 
- tools
twitter_text:  'exploring the app'
introduction: 'exploring the app'
---


# 前言


这篇使用 kubectl 来进行 APP 的故障排查

>In this scenario you will learn how to troubleshoot Kubernetes applications using the kubectl get, describe, logs and exec commands

参考 **[Exploring Your App][explore_interactive]** 


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

~~~
nothing@pc:~$ kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-69fff89cb6-42cpz   1/1     Running   0          79m
nothing@pc:~$ 
~~~

## 查看 pods 详细信息

~~~
nothing@pc:~$ kubectl describe pods
Name:           kubernetes-bootcamp-69fff89cb6-42cpz
Namespace:      default
Priority:       0
Node:           minikube/10.0.2.15
Start Time:     Tue, 13 Aug 2019 22:29:02 +0800
Labels:         pod-template-hash=69fff89cb6
                run=kubernetes-bootcamp
Annotations:    <none>
Status:         Running
IP:             172.17.0.5
Controlled By:  ReplicaSet/kubernetes-bootcamp-69fff89cb6
Containers:
  kubernetes-bootcamp:
    Container ID:   docker://c57050c4e5486a3a9732dc225086423411ff61d4db603b91b81433c5c46ca74b
    Image:          daocloud.io/library/nginx
    Image ID:       docker-pullable://daocloud.io/library/nginx@sha256:f83b2ffd963ac911f9e638184c8d580cc1f3139d5c8c33c87c3fb90aebdebf76
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 13 Aug 2019 22:30:52 +0800
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
Events:          <none>
nothing@pc:~$ 
~~~

>**Tip:** describe 命令可以用来获取 k8s 中大部分对象的详细信息

## 开启 proxy

~~~
nothing@pc:~$ echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n"; kubectl proxy



Starting Proxy. After starting it will not output a response. Please click the first Terminal Tab

Starting to serve on 127.0.0.1:8001
...
...
...
~~~

Pod 运行在一个隔离的私有环境下

我们要借用 Proxy 来构建一个到 pod 的通道，并且与之交互

我们获取一下 Pod 的名称


~~~
nothing@pc:~$ export POD_NAME=$(kubectl get pods -o go-template --template '{\{range .items}\}{\{.metadata.name}\}{\{"\n"}\}{\{end}\}')
nothing@pc:~$ echo Name of the Pod: $POD_NAME
Name of the Pod: kubernetes-bootcamp-69fff89cb6-42cpz
nothing@pc:~$ 
~~~

看一下应用的输出

~~~
nothing@pc:~$ curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
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

## 查看 Pod 的日志

~~~
nothing@pc:~$ kubectl logs $POD_NAME
172.17.0.1 - - [13/Aug/2019:15:08:19 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.61.0" "127.0.0.1, 192.168.99.1"
172.17.0.1 - - [13/Aug/2019:15:15:16 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.61.0" "127.0.0.1, 192.168.99.1"
172.17.0.1 - - [13/Aug/2019:15:59:13 +0000] "GET / HTTP/1.1" 200 612 "-" "curl/7.61.0" "127.0.0.1, 192.168.99.1"
nothing@pc:~$ 
~~~

一般而言应用的日志都输出到了 **STDOUT** 

我们可以通过 logs 来获取 Pod 中的日志

## 在容器中执行命令


~~~
nothing@pc:~$ kubectl exec $POD_NAME env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=kubernetes-bootcamp-69fff89cb6-42cpz
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_SERVICE_PORT=443
NGINX_VERSION=1.17.1
NJS_VERSION=0.3.3
PKG_RELEASE=1~stretch
HOME=/root
nothing@pc:~$
~~~

当 Pod 已经启动正在运行的状态，我们可以直接在容器中执行命令

当环境中只有一个 Pod 时，Pod 名是可以省略的

## 在容器中开启一个会话

~~~
nothing@pc:~$ kubectl exec -it $POD_NAME bash
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# ll  
bash: ll: command not found
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# exit
exit
command terminated with exit code 127
nothing@pc:~$ kubectl exec -it $POD_NAME bash
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# ls /etc/nginx/nginx.conf 
/etc/nginx/nginx.conf
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# ls /etc/nginx/conf.d/default.conf 
/etc/nginx/conf.d/default.conf
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# cat /usr/share/nginx/html/index.html
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
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# 
~~~

可以通过这种方式进入容器，在内部进行检查


然后在里面可以各种操作

~~~
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# apt install curl
Reading package lists... Done
Building dependency tree       
Reading state information... Done
E: Unable to locate package curl
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# apt-get update
Ign:2 http://cdn-fastly.deb.debian.org/debian stretch InRelease                                           
Get:3 http://cdn-fastly.deb.debian.org/debian stretch-updates InRelease [91.0 kB]
Get:1 http://security-cdn.debian.org/debian-security stretch/updates InRelease [94.3 kB]
Get:4 http://cdn-fastly.deb.debian.org/debian stretch Release [118 kB]                    
Get:5 http://cdn-fastly.deb.debian.org/debian stretch-updates/main amd64 Packages [27.4 kB]
Get:6 http://cdn-fastly.deb.debian.org/debian stretch Release.gpg [2434 B]                                                              
Get:7 http://security-cdn.debian.org/debian-security stretch/updates/main amd64 Packages [502 kB]                                       
Get:8 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 Packages [7082 kB]                                                     
Fetched 7918 kB in 1min 56s (68.2 kB/s)                                                                                                 
Reading package lists... Done
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# apt install curl
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  ca-certificates krb5-locales libcurl3 libffi6 libgmp10 libgnutls30 libgssapi-krb5-2 libhogweed4 libidn11 libidn2-0 libk5crypto3
  libkeyutils1 libkrb5-3 libkrb5support0 libldap-2.4-2 libldap-common libnettle6 libnghttp2-14 libp11-kit0 libpsl5 librtmp1 libsasl2-2
  libsasl2-modules libsasl2-modules-db libssh2-1 libssl1.0.2 libtasn1-6 libunistring0 openssl publicsuffix
Suggested packages:
  gnutls-bin krb5-doc krb5-user libsasl2-modules-gssapi-mit | libsasl2-modules-gssapi-heimdal libsasl2-modules-ldap
  libsasl2-modules-otp libsasl2-modules-sql
The following NEW packages will be installed:
  ca-certificates curl krb5-locales libcurl3 libffi6 libgmp10 libgnutls30 libgssapi-krb5-2 libhogweed4 libidn11 libidn2-0 libk5crypto3
  libkeyutils1 libkrb5-3 libkrb5support0 libldap-2.4-2 libldap-common libnettle6 libnghttp2-14 libp11-kit0 libpsl5 librtmp1 libsasl2-2
  libsasl2-modules libsasl2-modules-db libssh2-1 libssl1.0.2 libtasn1-6 libunistring0 openssl publicsuffix
0 upgraded, 31 newly installed, 0 to remove and 1 not upgraded.
Need to get 6626 kB of archives.
After this operation, 16.9 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
Get:1 http://security-cdn.debian.org/debian-security stretch/updates/main amd64 libssl1.0.2 amd64 1.0.2s-1~deb9u1 [1302 kB]
Get:3 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 krb5-locales all 1.15-1+deb9u1 [93.8 kB]              
Get:4 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libgmp10 amd64 2:6.1.2+dfsg-1 [253 kB]               
Get:5 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libnettle6 amd64 3.3-1+b2 [192 kB]
Get:6 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libhogweed4 amd64 3.3-1+b2 [136 kB]                                    
Get:7 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libidn11 amd64 1.33-1 [115 kB]                                         
Get:8 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libffi6 amd64 3.2.1-6 [20.4 kB]                                        
Get:9 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libp11-kit0 amd64 0.23.3-2 [111 kB]                                    
Get:10 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libtasn1-6 amd64 4.10-1.1+deb9u1 [50.6 kB]                            
Get:11 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libgnutls30 amd64 3.5.8-5+deb9u4 [896 kB]                             
Get:2 http://security-cdn.debian.org/debian-security stretch/updates/main amd64 openssl amd64 1.1.0k-1~deb9u1 [747 kB]                  
Get:12 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libkeyutils1 amd64 1.5.9-9 [12.4 kB]                                  
Get:13 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libkrb5support0 amd64 1.15-1+deb9u1 [61.9 kB]                         
Get:14 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libk5crypto3 amd64 1.15-1+deb9u1 [119 kB]                             
Get:15 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libkrb5-3 amd64 1.15-1+deb9u1 [311 kB]                                
Get:16 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libgssapi-krb5-2 amd64 1.15-1+deb9u1 [155 kB]                         
Get:17 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libsasl2-modules-db amd64 2.1.27~101-g0780600+dfsg-3 [68.2 kB]        
Get:18 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libsasl2-2 amd64 2.1.27~101-g0780600+dfsg-3 [105 kB]                  
Get:19 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libldap-common all 2.4.44+dfsg-5+deb9u2 [85.5 kB]                     
Get:20 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libldap-2.4-2 amd64 2.4.44+dfsg-5+deb9u2 [219 kB]                     
Get:21 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 ca-certificates all 20161130+nmu1+deb9u1 [182 kB]                     
Get:22 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libunistring0 amd64 0.9.6+really0.9.3-0.1 [279 kB]                    
Get:23 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libidn2-0 amd64 0.16-1+deb9u1 [60.7 kB]                               
Get:24 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libnghttp2-14 amd64 1.18.1-1 [79.1 kB]                                
Get:25 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libpsl5 amd64 0.17.0-3 [41.8 kB]                                      
Get:26 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 librtmp1 amd64 2.4+20151223.gitfa8646d.1-1+b1 [60.4 kB]               
Get:27 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libssh2-1 amd64 1.7.0-1+deb9u1 [139 kB]                               
Get:28 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libcurl3 amd64 7.52.1-5+deb9u9 [292 kB]                               
Get:29 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 curl amd64 7.52.1-5+deb9u9 [227 kB]                                   
Get:30 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 libsasl2-modules amd64 2.1.27~101-g0780600+dfsg-3 [102 kB]            
Get:31 http://cdn-fastly.deb.debian.org/debian stretch/main amd64 publicsuffix all 20190415.1030-0+deb9u1 [108 kB]                      
Fetched 6626 kB in 1min 31s (72.6 kB/s)                                                                                                 
debconf: delaying package configuration, since apt-utils is not installed
Selecting previously unselected package libssl1.0.2:amd64.
(Reading database ... 7034 files and directories currently installed.)
Preparing to unpack .../00-libssl1.0.2_1.0.2s-1~deb9u1_amd64.deb ...
Unpacking libssl1.0.2:amd64 (1.0.2s-1~deb9u1) ...
Selecting previously unselected package krb5-locales.
Preparing to unpack .../01-krb5-locales_1.15-1+deb9u1_all.deb ...
Unpacking krb5-locales (1.15-1+deb9u1) ...
Selecting previously unselected package libgmp10:amd64.
Preparing to unpack .../02-libgmp10_2%3a6.1.2+dfsg-1_amd64.deb ...
Unpacking libgmp10:amd64 (2:6.1.2+dfsg-1) ...
Selecting previously unselected package libnettle6:amd64.
Preparing to unpack .../03-libnettle6_3.3-1+b2_amd64.deb ...
Unpacking libnettle6:amd64 (3.3-1+b2) ...
Selecting previously unselected package libhogweed4:amd64.
Preparing to unpack .../04-libhogweed4_3.3-1+b2_amd64.deb ...
Unpacking libhogweed4:amd64 (3.3-1+b2) ...
Selecting previously unselected package libidn11:amd64.
Preparing to unpack .../05-libidn11_1.33-1_amd64.deb ...
Unpacking libidn11:amd64 (1.33-1) ...
Selecting previously unselected package libffi6:amd64.
Preparing to unpack .../06-libffi6_3.2.1-6_amd64.deb ...
Unpacking libffi6:amd64 (3.2.1-6) ...
Selecting previously unselected package libp11-kit0:amd64.
Preparing to unpack .../07-libp11-kit0_0.23.3-2_amd64.deb ...
Unpacking libp11-kit0:amd64 (0.23.3-2) ...
Selecting previously unselected package libtasn1-6:amd64.
Preparing to unpack .../08-libtasn1-6_4.10-1.1+deb9u1_amd64.deb ...
Unpacking libtasn1-6:amd64 (4.10-1.1+deb9u1) ...
Selecting previously unselected package libgnutls30:amd64.
Preparing to unpack .../09-libgnutls30_3.5.8-5+deb9u4_amd64.deb ...
Unpacking libgnutls30:amd64 (3.5.8-5+deb9u4) ...
Selecting previously unselected package libkeyutils1:amd64.
Preparing to unpack .../10-libkeyutils1_1.5.9-9_amd64.deb ...
Unpacking libkeyutils1:amd64 (1.5.9-9) ...
Selecting previously unselected package libkrb5support0:amd64.
Preparing to unpack .../11-libkrb5support0_1.15-1+deb9u1_amd64.deb ...
Unpacking libkrb5support0:amd64 (1.15-1+deb9u1) ...
Selecting previously unselected package libk5crypto3:amd64.
Preparing to unpack .../12-libk5crypto3_1.15-1+deb9u1_amd64.deb ...
Unpacking libk5crypto3:amd64 (1.15-1+deb9u1) ...
Selecting previously unselected package libkrb5-3:amd64.
Preparing to unpack .../13-libkrb5-3_1.15-1+deb9u1_amd64.deb ...
Unpacking libkrb5-3:amd64 (1.15-1+deb9u1) ...
Selecting previously unselected package libgssapi-krb5-2:amd64.
Preparing to unpack .../14-libgssapi-krb5-2_1.15-1+deb9u1_amd64.deb ...
Unpacking libgssapi-krb5-2:amd64 (1.15-1+deb9u1) ...
Selecting previously unselected package libsasl2-modules-db:amd64.
Preparing to unpack .../15-libsasl2-modules-db_2.1.27~101-g0780600+dfsg-3_amd64.deb ...
Unpacking libsasl2-modules-db:amd64 (2.1.27~101-g0780600+dfsg-3) ...
Selecting previously unselected package libsasl2-2:amd64.
Preparing to unpack .../16-libsasl2-2_2.1.27~101-g0780600+dfsg-3_amd64.deb ...
Unpacking libsasl2-2:amd64 (2.1.27~101-g0780600+dfsg-3) ...
Selecting previously unselected package libldap-common.
Preparing to unpack .../17-libldap-common_2.4.44+dfsg-5+deb9u2_all.deb ...
Unpacking libldap-common (2.4.44+dfsg-5+deb9u2) ...
Selecting previously unselected package libldap-2.4-2:amd64.
Preparing to unpack .../18-libldap-2.4-2_2.4.44+dfsg-5+deb9u2_amd64.deb ...
Unpacking libldap-2.4-2:amd64 (2.4.44+dfsg-5+deb9u2) ...
Selecting previously unselected package openssl.
Preparing to unpack .../19-openssl_1.1.0k-1~deb9u1_amd64.deb ...
Unpacking openssl (1.1.0k-1~deb9u1) ...
Selecting previously unselected package ca-certificates.
Preparing to unpack .../20-ca-certificates_20161130+nmu1+deb9u1_all.deb ...
Unpacking ca-certificates (20161130+nmu1+deb9u1) ...
Selecting previously unselected package libunistring0:amd64.
Preparing to unpack .../21-libunistring0_0.9.6+really0.9.3-0.1_amd64.deb ...
Unpacking libunistring0:amd64 (0.9.6+really0.9.3-0.1) ...
Selecting previously unselected package libidn2-0:amd64.
Preparing to unpack .../22-libidn2-0_0.16-1+deb9u1_amd64.deb ...
Unpacking libidn2-0:amd64 (0.16-1+deb9u1) ...
Selecting previously unselected package libnghttp2-14:amd64.
Preparing to unpack .../23-libnghttp2-14_1.18.1-1_amd64.deb ...
Unpacking libnghttp2-14:amd64 (1.18.1-1) ...
Selecting previously unselected package libpsl5:amd64.
Preparing to unpack .../24-libpsl5_0.17.0-3_amd64.deb ...
Unpacking libpsl5:amd64 (0.17.0-3) ...
Selecting previously unselected package librtmp1:amd64.
Preparing to unpack .../25-librtmp1_2.4+20151223.gitfa8646d.1-1+b1_amd64.deb ...
Unpacking librtmp1:amd64 (2.4+20151223.gitfa8646d.1-1+b1) ...
Selecting previously unselected package libssh2-1:amd64.
Preparing to unpack .../26-libssh2-1_1.7.0-1+deb9u1_amd64.deb ...
Unpacking libssh2-1:amd64 (1.7.0-1+deb9u1) ...
Selecting previously unselected package libcurl3:amd64.
Preparing to unpack .../27-libcurl3_7.52.1-5+deb9u9_amd64.deb ...
Unpacking libcurl3:amd64 (7.52.1-5+deb9u9) ...
Selecting previously unselected package curl.
Preparing to unpack .../28-curl_7.52.1-5+deb9u9_amd64.deb ...
Unpacking curl (7.52.1-5+deb9u9) ...
Selecting previously unselected package libsasl2-modules:amd64.
Preparing to unpack .../29-libsasl2-modules_2.1.27~101-g0780600+dfsg-3_amd64.deb ...
Unpacking libsasl2-modules:amd64 (2.1.27~101-g0780600+dfsg-3) ...
Selecting previously unselected package publicsuffix.
Preparing to unpack .../30-publicsuffix_20190415.1030-0+deb9u1_all.deb ...
Unpacking publicsuffix (20190415.1030-0+deb9u1) ...
Setting up libnettle6:amd64 (3.3-1+b2) ...
Setting up libnghttp2-14:amd64 (1.18.1-1) ...
Setting up libldap-common (2.4.44+dfsg-5+deb9u2) ...
Setting up libsasl2-modules-db:amd64 (2.1.27~101-g0780600+dfsg-3) ...
Setting up libsasl2-2:amd64 (2.1.27~101-g0780600+dfsg-3) ...
Setting up libtasn1-6:amd64 (4.10-1.1+deb9u1) ...
Setting up libssl1.0.2:amd64 (1.0.2s-1~deb9u1) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 76.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.24.1 /usr/local/share/perl/5.24.1 /usr/lib/x86_64-linux-gnu/perl5/5.24 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.24 /usr/share/perl/5.24 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base .) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Setting up libgmp10:amd64 (2:6.1.2+dfsg-1) ...
Setting up libssh2-1:amd64 (1.7.0-1+deb9u1) ...
Setting up krb5-locales (1.15-1+deb9u1) ...
Processing triggers for libc-bin (2.24-11+deb9u4) ...
Setting up publicsuffix (20190415.1030-0+deb9u1) ...
Setting up libunistring0:amd64 (0.9.6+really0.9.3-0.1) ...
Setting up openssl (1.1.0k-1~deb9u1) ...
Setting up libffi6:amd64 (3.2.1-6) ...
Setting up libkeyutils1:amd64 (1.5.9-9) ...
Setting up libsasl2-modules:amd64 (2.1.27~101-g0780600+dfsg-3) ...
Setting up ca-certificates (20161130+nmu1+deb9u1) ...
debconf: unable to initialize frontend: Dialog
debconf: (No usable dialog-like program is installed, so the dialog based frontend cannot be used. at /usr/share/perl5/Debconf/FrontEnd/Dialog.pm line 76.)
debconf: falling back to frontend: Readline
debconf: unable to initialize frontend: Readline
debconf: (Can't locate Term/ReadLine.pm in @INC (you may need to install the Term::ReadLine module) (@INC contains: /etc/perl /usr/local/lib/x86_64-linux-gnu/perl/5.24.1 /usr/local/share/perl/5.24.1 /usr/lib/x86_64-linux-gnu/perl5/5.24 /usr/share/perl5 /usr/lib/x86_64-linux-gnu/perl/5.24 /usr/share/perl/5.24 /usr/local/lib/site_perl /usr/lib/x86_64-linux-gnu/perl-base .) at /usr/share/perl5/Debconf/FrontEnd/Readline.pm line 7.)
debconf: falling back to frontend: Teletype
Updating certificates in /etc/ssl/certs...
151 added, 0 removed; done.
Setting up libidn11:amd64 (1.33-1) ...
Setting up libidn2-0:amd64 (0.16-1+deb9u1) ...
Setting up libpsl5:amd64 (0.17.0-3) ...
Setting up libkrb5support0:amd64 (1.15-1+deb9u1) ...
Setting up libhogweed4:amd64 (3.3-1+b2) ...
Setting up libp11-kit0:amd64 (0.23.3-2) ...
Setting up libk5crypto3:amd64 (1.15-1+deb9u1) ...
Setting up libgnutls30:amd64 (3.5.8-5+deb9u4) ...
Setting up librtmp1:amd64 (2.4+20151223.gitfa8646d.1-1+b1) ...
Setting up libldap-2.4-2:amd64 (2.4.44+dfsg-5+deb9u2) ...
Setting up libkrb5-3:amd64 (1.15-1+deb9u1) ...
Setting up libgssapi-krb5-2:amd64 (1.15-1+deb9u1) ...
Setting up libcurl3:amd64 (7.52.1-5+deb9u9) ...
Setting up curl (7.52.1-5+deb9u9) ...
Processing triggers for libc-bin (2.24-11+deb9u4) ...
Processing triggers for ca-certificates (20161130+nmu1+deb9u1) ...
Updating certificates in /etc/ssl/certs...
0 added, 0 removed; done.
Running hooks in /etc/ca-certificates/update.d...
done.
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# 
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# 
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# curl localhost:80
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
root@kubernetes-bootcamp-69fff89cb6-42cpz:/# 
~~~



---

# 总结

主要是 kubectl 命令的使用

~~~
kubectl get pods
kubectl describe pods
export POD_NAME=$(kubectl get pods -o go-template --template '{\{range .items}\}{\{.metadata.name}\}{\{"\n"}\}{\{end}\}')
echo Name of the Pod: $POD_NAME
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
kubectl logs $POD_NAME
kubectl exec $POD_NAME env
kubectl exec -it $POD_NAME bash
~~~

* TOC
{:toc}

---


[explore_interactive]:https://kubernetes.io/docs/tutorials/kubernetes-basics/explore/explore-interactive/
[kubernetes]:https://kubernetes.io/


