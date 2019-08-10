---
layout: post
title: "Zabbix Datasource in  Grafana "
date: 2019-08-06 14:29:07
image: '/assets/img/'
description:  'Grafana 上添加 Zabbix Datasource'
main-class:  'grafana'
color: 
tags:
    - grafana
    - zabbix
categories:
    - grafana
twitter_text: 'simple process of zabbix Datasource for grafana configration'
introduction: 'simple process of zabbix Datasource for grafana configration'
---



## 前言

**[Zabbix Plugin for Grafana][alexanderzobnin-zabbix-app]** 是一个Grafana 的插件

用来将 Zabbix  中的数据拓展到 Grafana 中进行展示

>Visualize your Zabbix metrics with the leading open source software for time series analytics.

这里分享一下 **[Zabbix Plugin for Grafana][alexanderzobnin-zabbix-app]** 的安装方法

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



## 安装插件


先查看一下系统中已经安装了哪些插件

~~~
[root@grafana ~]# grafana-cli plugins ls
installed plugins:
grafana-worldmap-panel @ 0.2.0 

Restart grafana after installing plugins . <service grafana-server restart>

[root@grafana ~]# 
~~~

![grafana_plugin](/assets/img/grafana/grafana_plugin01.png)

此插件依赖 Grafana 3.0 及以上的版本

>**Note:** Grafana 3.0 or greater is required to install and use plugins

进行插件安装

~~~
[root@grafana ~]# time grafana-cli plugins install alexanderzobnin-zabbix-app
installing alexanderzobnin-zabbix-app @ 3.10.3
from url: https://grafana.com/api/plugins/alexanderzobnin-zabbix-app/versions/3.10.3/download
into: /var/lib/grafana/plugins

✔ Installed alexanderzobnin-zabbix-app successfully 

Restart grafana after installing plugins . <service grafana-server restart>


real	4m32.726s
user	0m0.163s
sys	0m0.350s
[root@grafana ~]# grafana-cli plugins ls
installed plugins:
alexanderzobnin-zabbix-app @ 3.10.3 
grafana-worldmap-panel @ 0.2.0 

Restart grafana after installing plugins . <service grafana-server restart>

[root@grafana ~]# 
~~~

插件目录中多了一个文件夹

~~~
[root@grafana ~]# ll /var/lib/grafana/plugins/
total 8
drwxr-xr-x. 8 root root 4096 Aug  6 13:59 alexanderzobnin-zabbix-app
drwxr-xr-x. 6 root root 4096 Jul 19 13:49 grafana-worldmap-panel
[root@grafana ~]# 
~~~



## 重启 grafana 服务


~~~
[root@grafana ~]# service grafana-server restart
Restarting grafana-server (via systemctl):                 [  OK  ]
[root@grafana ~]# 
~~~

多了一个插件

![grafana_plugin](/assets/img/grafana/grafana_plugin02.png)


## 启用插件

点击 Enable now 

![grafana_plugin](/assets/img/grafana/grafana_plugin03.png)


![grafana_plugin](/assets/img/grafana/grafana_plugin04.png)


---

# 总结

因为 Grafana 非常友好的插件扩展机制 **[Zabbix Plugin for Grafana][alexanderzobnin-zabbix-app]**  的安装非常容易

`**grafana-cli plugins install alexanderzobnin-zabbix-app**`


* TOC
{:toc}

---

[alexanderzobnin-zabbix-app]:https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app