---
layout: post
title: "Auto install gitea by compose"
date: 2020-09-11 02:37:42
image: '/assets/img/'
description: '自动安装 gitea'
main-class: 'code'
color: '#fbbd04' 
tags:
 - snippet
 - docker
 - tools
 - gitea
 - devops
 - code
categories:
 - code
twitter_text: "Auto install gitea by compose"
introduction: "Auto install gitea by compose"
---

# 前言

开启一个 **CODE** 系列，用来收集各种好用的小片段

这一篇用来展示如何通过 docker-compose 来安装 gitea

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
[root@docker ~]# hostnamectl
   Static hostname: docker
         Icon name: computer-vm
           Chassis: vm
        Machine ID: c7773881ade544df94677ad643d356d5
           Boot ID: 4c51606c39704637aeb61b9c8e4713b1
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@docker ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 82914sec preferred_lft 82914sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:7c:d7:94 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.210/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe7c:d794/64 scope link
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:12:18:a5:d1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft fo
~~~

## 代码

~~~
[root@docker gitea]# pwd
/docker/gitea
[root@docker gitea]# tree
.
├── docker-compose.yaml
└── init_gitea.bash

0 directories, 2 files
[root@docker gitea]# cat init_gitea.bash
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
docker pull gitea/gitea

## service up
docker-compose -f docker-compose.yaml  up -d
[root@docker gitea]# cat docker-compose.yaml
version: "3"

services:
  web:
    image: gitea/gitea:1.8.3
    ports:
      - "3000:3000"
      - "10022:22"
    networks:
      - giteanet
    restart: always
    volumes:
      - gitea_data:/data

networks:
  giteanet:
    driver: bridge

volumes:
  gitea_data:
[root@docker gitea]#
~~~



## 执行

~~~
[root@docker gitea]# time bash -x init_gitea.bash
+ '[' '!' -e /usr/local/bin/docker-compose ']'
+ docker pull gitea/gitea
Using default tag: latest
Trying to pull repository docker.io/gitea/gitea ...
latest: Pulling from docker.io/gitea/gitea
Digest: sha256:e851437e5907c0706abcd90594d4663f5c5388cd5a6a89ec4215e6f8cfd4e8bd
Status: Image is up to date for docker.io/gitea/gitea:latest
+ docker-compose -f docker-compose.yaml up -d
Creating network "gitea_giteanet" with driver "bridge"
Creating gitea_web_1 ... done

real	0m21.629s
user	0m0.410s
sys	0m0.077s
[root@docker gitea]#
[root@docker gitea]# docker-compose ps
   Name                  Command               State                       Ports
----------------------------------------------------------------------------------------------------
gitea_web_1   /usr/bin/entrypoint /bin/s ...   Up      0.0.0.0:10022->22/tcp, 0.0.0.0:3000->3000/tcp
[root@docker gitea]#
~~~


## 访问

访问

![gitea](/assets/img/gitea/gitea10.png)

配置，使用自带数据库

![gitea](/assets/img/gitea/gitea11.png)

![gitea](/assets/img/gitea/gitea12.png)

配置一个管理账户

![gitea](/assets/img/gitea/gitea13.png)

进行访问

![gitea](/assets/img/gitea/gitea14.png)


---

# 总结

通过 docker-compose 可以快速组建服务

是一种轻量的的编排工具，非常好用

* TOC
{:toc}

---
