---
layout: post
title: "Auto install Jenkins by compose"
date: 2020-09-11 04:37:42
image: '/assets/img/'
description: '自动安装 Jenkins'
main-class: 'code'
color: '#fbbd04' 
tags:
 - snippet
 - docker
 - tools
 - jenkins
 - devops
 - code
categories:
 - code
twitter_text: "Auto install Jenkins by compose"
introduction: "Auto install Jenkins by compose"
---

# 前言

开启一个 **CODE** 系列，用来收集各种好用的小片段

这一篇用来展示如何通过 docker-compose 来安装 Jenkins

> **Tip:** 展示中使用的各种软件版本可能不是最新，但我都会尽量提前交代

---

# 操作

## 依赖

* Docker 

~~~
[root@docker ~]# docker --version
Docker version 1.13.1, build 64e9980/1.13.1
[root@docker ~]#
~~~

## OS 环境

~~~
[root@docker jenkins]# hostnamectl
   Static hostname: docker
         Icon name: computer-vm
           Chassis: vm
        Machine ID: c7773881ade544df94677ad643d356d5
           Boot ID: d843b70cabeb47c199404beca1e9fa6f
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@docker jenkins]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 83090sec preferred_lft 83090sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:7c:d7:94 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.210/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe7c:d794/64 scope link
       valid_lft forever preferred_lft forever
4: br-0fca505096c3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:2f:5a:64:c3 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 scope global br-0fca505096c3
       valid_lft forever preferred_lft forever
5: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:da:4b:77:f3 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
9: br-7c1de2fbf9b9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:e4:7c:3a:c1 brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.1/16 scope global br-7c1de2fbf9b9
       valid_lft forever preferred_lft forever
    inet6 fe80::42:e4ff:fe7c:3ac1/64 scope link
       valid_lft forever preferred_lft forever
11: veth0b99d42@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-7c1de2fbf9b9 state UP group default
    link/ether 2a:30:dd:22:e4:88 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::2830:ddff:fe22:e488/64 scope link
       valid_lft forever preferred_lft forever
12: br-5b74f9491449: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:b2:d5:d7:c5 brd ff:ff:ff:ff:ff:ff
    inet 172.20.0.1/16 scope global br-5b74f9491449
       valid_lft forever preferred_lft forever
    inet6 fe80::42:b2ff:fed5:d7c5/64 scope link
       valid_lft forever preferred_lft forever
14: veth9ecde87@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-5b74f9491449 state UP group default
    link/ether 36:ff:4e:54:8c:2a brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::34ff:4eff:fe54:8c2a/64 scope link
       valid_lft forever preferred_lft forever
[root@docker jenkins]#
~~~

## 代码

~~~
[root@docker jenkins]# pwd
/docker/jenkins
[root@docker jenkins]# tree
.
├── docker-compose.yaml
└── init_jenkins.bash

0 directories, 2 files
[root@docker jenkins]# cat init_jenkins.bash
#!/bin/bash

## ensure compose
if [ ! -e /usr/local/bin/docker-compose ]; then
  ## download docker compose
  compose_version='1.27.0'
  ## sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
  sudo curl -L https://get.daocloud.io/docker/compose/releases/download/${compose_version}/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
  sudo chmod +x /usr/local/bin/docker-compose
  docker-compose --version
fi

## pull image
docker pull jenkins:2.60.3

## service up
docker-compose -f docker-compose.yaml  up -d
[root@docker jenkins]# cat docker-compose.yaml
version: "3"

services:
  web:
    image: jenkins:2.60.3
    ports:
      - "8080:8080"
      - "50000:50000"
    networks:
      - jenkinsnet
    restart: always
    volumes:
      - jenkins_home:/var/jenkins_home

networks:
  jenkinsnet:
    driver: bridge

volumes:
  jenkins_home:
[root@docker jenkins]#
~~~



## 执行

~~~
[root@docker jenkins]# time bash +x init_jenkins.bash
Trying to pull repository docker.io/library/jenkins ...
2.60.3: Pulling from docker.io/library/jenkins
55cbf04beb70: Pull complete
1607093a898c: Pull complete
9a8ea045c926: Pull complete
d4eee24d4dac: Pull complete
c58988e753d7: Pull complete
794a04897db9: Pull complete
70fcfa476f73: Pull complete
0539c80a02be: Pull complete
54fefc6dcf80: Pull complete
911bc90e47a8: Pull complete
38430d93efed: Pull complete
7e46ccda148a: Pull complete
c0cbcb5ac747: Pull complete
35ade7a86a8e: Pull complete
aa433a6a56b1: Pull complete
841c1dd38d62: Pull complete
b865dcb08714: Pull complete
5a3779030005: Pull complete
12b47c68955c: Pull complete
1322ea3e7bfd: Pull complete
Digest: sha256:eeb4850eb65f2d92500e421b430ed1ec58a7ac909e91f518926e02473904f668
Status: Downloaded newer image for docker.io/jenkins:2.60.3
Creating network "jenkins_jenkinsnet" with driver "bridge"
Creating volume "jenkins_jenkins_home" with default driver
Creating jenkins_web_1 ... done

real	1m30.587s
user	0m0.626s
sys	0m0.187s
[root@docker jenkins]#
[root@docker jenkins]# docker-compose ps
    Name                   Command               State                        Ports
---------------------------------------------------------------------------------------------------------
jenkins_web_1   /bin/tini -- /usr/local/bi ...   Up      0.0.0.0:50000->50000/tcp, 0.0.0.0:8080->8080/tcp
[root@docker jenkins]#
~~~


## 访问

根据提示输入密码

![jenkins](/assets/img/jenkins/jenkins45.png)

指定要安装的插件

![jenkins](/assets/img/jenkins/jenkins46.png)

![jenkins](/assets/img/jenkins/jenkins47.png)

配置管理用户

![jenkins](/assets/img/jenkins/jenkins48.png)

投入使用

![jenkins](/assets/img/jenkins/jenkins49.png)

不过在管理界面我们可以看到很多问题，这些问题需要通过升级 jenkins.war 来解决

![jenkins](/assets/img/jenkins/jenkins50.png)

最简单的办法是下载替换重启服务

---

# 总结

通过 docker-compose 可以快速组建服务

是一种轻量的的编排工具，非常好用

* TOC
{:toc}

---
