---
layout: post
title: "Little Traefik Demo"
date: 2020-09-23 09:00:00
image: '/assets/img/'
description: '通过 compose 构建 traefik'
main-class: 'code'
color: '#fbbd04' 
tags:
 - snippet
 - traefic
 - code
categories:
 - code
twitter_text: "Little Traefik Demo"
introduction: "Little Traefik Demo"
---

# 前言

开启一个 **[CODE][code]** 系列，用来收集各种好用的小片段

这一篇用来展示如何构建一个小的 Traefik Demo

> **Tip:** 此次使用的 traefic 版本为 **Traefik v2.2**

---

# 操作


## OS 环境

~~~
[root@docker traefic]# hostnamectl
   Static hostname: docker
         Icon name: computer-vm
           Chassis: vm
        Machine ID: c7773881ade544df94677ad643d356d5
           Boot ID: b91c4d43cd9b4c9ca3078f5f46fd95b5
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@docker traefic]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85000sec preferred_lft 85000sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:7c:d7:94 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.210/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe7c:d794/64 scope link
       valid_lft forever preferred_lft forever
4: br-0fca505096c3: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:6f:e9:fe:0e brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 scope global br-0fca505096c3
       valid_lft forever preferred_lft forever
5: br-5b74f9491449: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:5f:7b:a5:42 brd ff:ff:ff:ff:ff:ff
    inet 172.20.0.1/16 scope global br-5b74f9491449
       valid_lft forever preferred_lft forever
    inet6 fe80::42:5fff:fe7b:a542/64 scope link
       valid_lft forever preferred_lft forever
6: br-7c1de2fbf9b9: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:4e:28:14:89 brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.1/16 scope global br-7c1de2fbf9b9
       valid_lft forever preferred_lft forever
    inet6 fe80::42:4eff:fe28:1489/64 scope link
       valid_lft forever preferred_lft forever
7: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:57:b4:a2:cc brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
40: br-50f1d7858f60: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:b4:1e:21:79 brd ff:ff:ff:ff:ff:ff
    inet 172.21.0.1/16 scope global br-50f1d7858f60
       valid_lft forever preferred_lft forever
    inet6 fe80::42:b4ff:fe1e:2179/64 scope link
       valid_lft forever preferred_lft forever
42: veth6310561@if41: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-50f1d7858f60 state UP group default
    link/ether 32:d3:37:85:cb:f4 brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::30d3:37ff:fe85:cbf4/64 scope link
       valid_lft forever preferred_lft forever
44: veth1001ede@if43: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-50f1d7858f60 state UP group default
    link/ether 42:43:a3:64:32:ba brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::4043:a3ff:fe64:32ba/64 scope link
       valid_lft forever preferred_lft forever
46: vethde68c80@if45: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-50f1d7858f60 state UP group default
    link/ether 9a:bd:5e:86:3a:f4 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::98bd:5eff:fe86:3af4/64 scope link
       valid_lft forever preferred_lft forever
58: vethb93e5d7@if57: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-50f1d7858f60 state UP group default
    link/ether 16:91:11:b1:b3:3b brd ff:ff:ff:ff:ff:ff link-netnsid 3
    inet6 fe80::1491:11ff:feb1:b33b/64 scope link
       valid_lft forever preferred_lft forever
60: vethb2879b4@if59: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-50f1d7858f60 state UP group default
    link/ether fe:ce:b9:4e:a8:08 brd ff:ff:ff:ff:ff:ff link-netnsid 6
    inet6 fe80::fcce:b9ff:fe4e:a808/64 scope link
       valid_lft forever preferred_lft forever
62: veth0d88733@if61: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-50f1d7858f60 state UP group default
    link/ether 3e:0d:ee:14:71:e8 brd ff:ff:ff:ff:ff:ff link-netnsid 5
    inet6 fe80::3c0d:eeff:fe14:71e8/64 scope link
       valid_lft forever preferred_lft forever
64: veth9fe3234@if63: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-50f1d7858f60 state UP group default
    link/ether 22:05:d0:48:e2:33 brd ff:ff:ff:ff:ff:ff link-netnsid 7
    inet6 fe80::2005:d0ff:fe48:e233/64 scope link
       valid_lft forever preferred_lft forever
66: vethbbcd4cd@if65: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-50f1d7858f60 state UP group default
    link/ether 9e:b6:74:0d:bf:91 brd ff:ff:ff:ff:ff:ff link-netnsid 4
    inet6 fe80::9cb6:74ff:fe0d:bf91/64 scope link
       valid_lft forever preferred_lft forever
[root@docker traefic]#
~~~


## 依赖

docker compose 

~~~
[root@docker traefic]# docker --version
Docker version 1.13.1, build 64e9980/1.13.1
[root@docker traefic]# docker-compose --version
docker-compose version 1.27.0, build 980ec85b
[root@docker traefic]#
~~~


## 代码

~~~
[root@docker traefic]# cat docker-compose.yaml
version: '3'

networks:
  traefiknet:
    driver: bridge

services:
  traefik:
    image: traefik:v2.2
    command: --api.insecure=true --providers.docker
    networks:
      - traefiknet
    ports:
      - "80:80"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

  whoami:
    image: containous/whoami
    networks:
      - traefiknet
    labels:
      - "traefik.http.routers.whoami.rule=Host(`whoami.example.demo`)"
[root@docker traefic]#
~~~

这里打开了一个 **whoami** 的服务用于提供网络访问，内容为服务端收集到的信息

traefik 会自动发现这个服务，并且完成网络请求的路由

规则依据 header 中的 Host 进行判断

~~~
- "traefik.http.routers.whoami.rule=Host(`whoami.example.demo`)"
~~~


## 执行


~~~
[root@docker traefic]# docker-compose up -d
Creating network "traefic_traefiknet" with driver "bridge"
Creating traefic_whoami_1  ... done
Creating traefic_traefik_1 ... done
[root@docker traefic]#
[root@docker traefic]# docker-compose ps
      Name                     Command               State                     Ports
-------------------------------------------------------------------------------------------------------
traefic_traefik_1   /entrypoint.sh --api.insec ...   Up      0.0.0.0:80->80/tcp, 0.0.0.0:8080->8080/tcp
traefic_whoami_1    /whoami                          Up      80/tcp
[root@docker traefic]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                                        NAMES
720a91092afb        traefik:v2.2        "/entrypoint.sh --..."   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp, 0.0.0.0:8080->8080/tcp   traefic_traefik_1
74401de13e34        containous/whoami   "/whoami"                About a minute ago   Up About a minute   80/tcp                                       traefic_whoami_1
[root@docker traefic]#
[root@docker traefic]# curl -H Host:whoami.example.demo  http://127.0.0.1
Hostname: 74401de13e34
IP: 127.0.0.1
IP: ::1
IP: 172.21.0.2
IP: fe80::42:acff:fe15:2
RemoteAddr: 172.21.0.3:59530
GET / HTTP/1.1
Host: whoami.example.demo
User-Agent: curl/7.29.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.21.0.1
X-Forwarded-Host: whoami.example.demo
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 720a91092afb
X-Real-Ip: 172.21.0.1

[root@docker traefic]#
[root@docker traefic]# curl http://127.0.0.1
404 page not found
[root@docker traefic]#
~~~


拓展后端服务

~~~
[root@docker traefic]# docker-compose up --scale whoami=3 -d
Starting traefic_whoami_1 ...
Starting traefic_whoami_1 ... done
Creating traefic_whoami_2 ... done
Creating traefic_whoami_3 ... done
[root@docker traefic]# docker ps -a
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                        NAMES
71f811240e47        containous/whoami   "/whoami"                4 seconds ago       Up 4 seconds        80/tcp                                       traefic_whoami_2
6d7095cce201        containous/whoami   "/whoami"                4 seconds ago       Up 4 seconds        80/tcp                                       traefic_whoami_3
720a91092afb        traefik:v2.2        "/entrypoint.sh --..."   11 minutes ago      Up 11 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:8080->8080/tcp   traefic_traefik_1
74401de13e34        containous/whoami   "/whoami"                11 minutes ago      Up 11 minutes       80/tcp                                       traefic_whoami_1
[root@docker traefic]#
[root@docker traefic]# curl -H Host:whoami.example.demo  http://127.0.0.1
Hostname: 6d7095cce201
IP: 127.0.0.1
IP: ::1
IP: 172.21.0.4
IP: fe80::42:acff:fe15:4
RemoteAddr: 172.21.0.3:38082
GET / HTTP/1.1
Host: whoami.example.demo
User-Agent: curl/7.29.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.21.0.1
X-Forwarded-Host: whoami.example.demo
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 720a91092afb
X-Real-Ip: 172.21.0.1

[root@docker traefic]# curl -H Host:whoami.example.demo  http://127.0.0.1
Hostname: 71f811240e47
IP: 127.0.0.1
IP: ::1
IP: 172.21.0.5
IP: fe80::42:acff:fe15:5
RemoteAddr: 172.21.0.3:41664
GET / HTTP/1.1
Host: whoami.example.demo
User-Agent: curl/7.29.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.21.0.1
X-Forwarded-Host: whoami.example.demo
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 720a91092afb
X-Real-Ip: 172.21.0.1

[root@docker traefic]# curl -H Host:whoami.example.demo  http://127.0.0.1
Hostname: 74401de13e34
IP: 127.0.0.1
IP: ::1
IP: 172.21.0.2
IP: fe80::42:acff:fe15:2
RemoteAddr: 172.21.0.3:59558
GET / HTTP/1.1
Host: whoami.example.demo
User-Agent: curl/7.29.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.21.0.1
X-Forwarded-Host: whoami.example.demo
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 720a91092afb
X-Real-Ip: 172.21.0.1

[root@docker traefic]# curl -H Host:whoami.example.demo  http://127.0.0.1
Hostname: 6d7095cce201
IP: 127.0.0.1
IP: ::1
IP: 172.21.0.4
IP: fe80::42:acff:fe15:4
RemoteAddr: 172.21.0.3:38082
GET / HTTP/1.1
Host: whoami.example.demo
User-Agent: curl/7.29.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.21.0.1
X-Forwarded-Host: whoami.example.demo
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 720a91092afb
X-Real-Ip: 172.21.0.1

[root@docker traefic]# curl -H Host:whoami.example.demo  http://127.0.0.1
Hostname: 71f811240e47
IP: 127.0.0.1
IP: ::1
IP: 172.21.0.5
IP: fe80::42:acff:fe15:5
RemoteAddr: 172.21.0.3:41664
GET / HTTP/1.1
Host: whoami.example.demo
User-Agent: curl/7.29.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.21.0.1
X-Forwarded-Host: whoami.example.demo
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 720a91092afb
X-Real-Ip: 172.21.0.1

[root@docker traefic]# curl -H Host:whoami.example.demo  http://127.0.0.1
Hostname: 74401de13e34
IP: 127.0.0.1
IP: ::1
IP: 172.21.0.2
IP: fe80::42:acff:fe15:2
RemoteAddr: 172.21.0.3:59558
GET / HTTP/1.1
Host: whoami.example.demo
User-Agent: curl/7.29.0
Accept: */*
Accept-Encoding: gzip
X-Forwarded-For: 172.21.0.1
X-Forwarded-Host: whoami.example.demo
X-Forwarded-Port: 80
X-Forwarded-Proto: http
X-Forwarded-Server: 720a91092afb
X-Real-Ip: 172.21.0.1

[root@docker traefic]#
~~~
 
拓展完成后，可以看到再次访问，轮询到后面的服务器中

![traefik](/assets/img/post/2020-09-23-auto-install-traefik/traefik01.jpg)

这里可以看到后面的三个服务

后面可以演示如何对 traefic 进行进一步的配置

**https://raw.githubusercontent.com/containous/traefik/master/traefik.sample.toml**


此篇参考了 **[Traefik 简易入门][easy]** 中的内容

---

# 总结

通过 traefic 进行负载均衡，无须重启，会自动完成服务发现与负载均衡，基于 container label 配置，有着漂亮的 dashboard 界面


* TOC
{:toc}

---

[code]:http://www.atomicgain.com/category/code/
[easy]:https://shanyue.tech/op/traefik.html
