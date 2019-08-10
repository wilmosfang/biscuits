---
layout: post
title: "Zabbix DataSource in  Grafana "
date: 2019-08-10 14:29:07
image: '/assets/img/'
description:  'Grafana 上添加 Zabbix Datasource'
main-class:  'grafana'
color: 
tags:
    - grafana
    - zabbix
categories:
    - grafana
twitter_text: 'simple process of zabbix DataSource for grafana configration'
introduction: 'simple process of zabbix DataSource for grafana configration'
---



## 前言

**[Zabbix Plugin for Grafana][alexanderzobnin-zabbix-app]** 是一个Grafana 的插件

用来将 Zabbix  中的数据拓展到 Grafana 中进行展示

>Visualize your Zabbix metrics with the leading open source software for time series analytics.

这里分享一下 **[Zabbix Plugin for Grafana][alexanderzobnin-zabbix-app]** 的数据源添加方法

参考 **[Zabbix Plugin for Grafana][alexanderzobnin-zabbix-app]** 


> **Tip:** 当前的版本为 **3.10.3**

---

# 操作


## 环境

~~~
[root@grafana ~]# hostnamectl 
   Static hostname: grafana
         Icon name: computer-vm
           Chassis: vm
        Machine ID: a8128cfbdef948119ae7fb0f88b37ee7
           Boot ID: e1bc4caca613423f8993bd4a64658b14
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.1.3.el7.x86_64
      Architecture: x86-64
[root@grafana ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:84:81:d5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86302sec preferred_lft 86302sec
    inet6 fe80::5054:ff:fe84:81d5/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:a2:bc:94 brd ff:ff:ff:ff:ff:ff
    inet 10.1.0.51/24 brd 10.1.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea2:bc94/64 scope link 
       valid_lft forever preferred_lft forever
[root@grafana ~]# rpm -qa | grep grafana
grafana-6.2.5-1.x86_64
[root@grafana ~]# 
~~~


已经完成了插件的安装

![grafana_plugin](/assets/img/grafana/grafana_plugin02.png)

并且已经有一个正在运行中的 zabbix 

![grafana_plugin](/assets/img/grafana/grafana_plugin07.png)

## 添加数据源


![grafana_plugin](/assets/img/grafana/grafana_plugin08.png)

配置 MySQL

![grafana_plugin](/assets/img/grafana/grafana_plugin09.png)

![grafana_plugin](/assets/img/grafana/grafana_plugin10.png)

配置 Zabbix

![grafana_plugin](/assets/img/grafana/grafana_plugin11.png)

![grafana_plugin](/assets/img/grafana/grafana_plugin12.png)


## 应用数据源配置走势图

![grafana_plugin](/assets/img/grafana/grafana_plugin13.png)

对比 zabbix 原生的展示效果

![grafana_plugin](/assets/img/grafana/grafana_plugin14.png)

可见这颜值提升了不止一点


---

# 总结

因为 Grafana 非常友好的插件扩展机制 

**[Zabbix Plugin for Grafana][alexanderzobnin-zabbix-app]**  的配置非常容易

全程都可以在 web 控制台中完成


* TOC
{:toc}

---

[alexanderzobnin-zabbix-app]:https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app