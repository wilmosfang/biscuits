---
layout: post
title: "Install MediaWiki"
date: 2018-04-12 14:35:32
image: '/assets/img/'
description: '安装 MediaWiki'
main-class:  'tools'
color: '#808080'
tags:
 - tools
 - wiki
categories:
 - tools
twitter_text: 'simple process of MediaWiki installation'
introduction: 'Installation of MediaWiki'
---
# 前言

**[MediaWiki][mediawiki]** 是一款用 php 实现的开源 **wiki** 软件

>MediaWiki is a free software open source wiki package written in PHP, originally for use on Wikipedia.
>
>It is now also used by several other projects of the non-profit Wikimedia Foundation and by many other wikis, including this website, the home of MediaWiki.

作为一种知识共享的平台，**wiki** 在全世界都广受欢迎

使用 **[MediaWiki][mediawiki]** 可以在组织内部构建一个知识共享平台，用于知识分享

在技术型组织里，可以使用它来构建一个文档管理系统

这里演示一下如何构建 **[MediaWiki][mediawiki]**

参考 **[Running MediaWiki on Red Hat Linux][mediawiki_install]**

> **Tip:** 当前的版本为 **openvpn 2.4.5**

---

# 操作

## 环境

~~~bash
[root@wiki ~]# hostnamectl
   Static hostname: wiki
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: 04ccdae7f06f48508a95022fc4b1e986
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@wiki ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:f9:30:bb brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 82430sec preferred_lft 82430sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:a1:e7:17 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.210/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea1:e717/64 scope link tentative dadfailed 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@wiki ~]#
~~~

## 安装 centos-release-scl 库

~~~
[root@wiki ~]# yum install centos-release-scl
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: centos-hcm.viettelidc.com.vn
 * c7-media: 
 * epel: mirror.smartmedia.net.id
 * extras: centos-hcm.viettelidc.com.vn
 * updates: centos-hcm.viettelidc.com.vn
Resolving Dependencies
--> Running transaction check
---> Package centos-release-scl.noarch 0:2-2.el7.centos will be installed
--> Processing Dependency: centos-release-scl-rh for package: centos-release-scl-2-2.el7.centos.noarch
--> Running transaction check
---> Package centos-release-scl-rh.noarch 0:2-2.el7.centos will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                    Arch        Version               Repository   Size
================================================================================
Installing:
 centos-release-scl         noarch      2-2.el7.centos        extras       12 k
Installing for dependencies:
 centos-release-scl-rh      noarch      2-2.el7.centos        extras       12 k

Transaction Summary
================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 24 k
Installed size: 39 k
Is this ok [y/d/N]: y
Downloading packages:
(1/2): centos-release-scl-2-2.el7.centos.noarch.rpm        |  12 kB   00:00     
(2/2): centos-release-scl-rh-2-2.el7.centos.noarch.rpm     |  12 kB   00:00     
--------------------------------------------------------------------------------
Total                                               70 kB/s |  24 kB  00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
** Found 3 pre-existing rpmdb problem(s), 'yum check' output follows:
ipa-client-4.4.0-14.el7.centos.7.x86_64 has installed conflicts freeipa-client: ipa-client-4.4.0-14.el7.centos.7.x86_64
ipa-client-common-4.4.0-14.el7.centos.7.noarch has installed conflicts freeipa-client-common: ipa-client-common-4.4.0-14.el7.centos.7.noarch
ipa-common-4.4.0-14.el7.centos.7.noarch has installed conflicts freeipa-common: ipa-common-4.4.0-14.el7.centos.7.noarch
  Installing : centos-release-scl-rh-2-2.el7.centos.noarch                  1/2 
  Installing : centos-release-scl-2-2.el7.centos.noarch                     2/2 
  Verifying  : centos-release-scl-rh-2-2.el7.centos.noarch                  1/2 
  Verifying  : centos-release-scl-2-2.el7.centos.noarch                     2/2 

Installed:
  centos-release-scl.noarch 0:2-2.el7.centos                                    

Dependency Installed:
  centos-release-scl-rh.noarch 0:2-2.el7.centos                                 

Complete!
[root@wiki ~]# 
~~~

## 安装必要软件

**`yum install httpd24-httpd php55 php55-php php55-php-mbstring php55-php-mysqlnd php55-php-gd php55-php-xml mariadb-server mariadb`**

~~~
[root@wiki ~]# yum install httpd24-httpd php55 php55-php php55-php-mbstring php55-php-mysqlnd php55-php-gd php55-php-xml mariadb-server mariadb
Loaded plugins: fastestmirror, langpacks
centos-sclo-rh                                           | 3.0 kB     00:00     
centos-sclo-sclo                                         | 2.9 kB     00:00     
(1/2): centos-sclo-sclo/x86_64/primary_db                  | 202 kB   00:06     
(2/2): centos-sclo-rh/x86_64/primary_db                    | 3.2 MB   00:13     
Loading mirror speeds from cached hostfile
 * base: centos-hcm.viettelidc.com.vn
 * c7-media: 
 * epel: ftp.riken.jp
 * extras: centos-hcm.viettelidc.com.vn
 * updates: centos-hcm.viettelidc.com.vn
Resolving Dependencies
--> Running transaction check
---> Package httpd24-httpd.x86_64 0:2.4.27-8.el7 will be installed
--> Processing Dependency: httpd24-httpd-tools = 2.4.27-8.el7 for package: httpd24-httpd-2.4.27-8.el7.x86_64
--> Processing Dependency: httpd24-runtime for package: httpd24-httpd-2.4.27-8.el7.x86_64
--> Processing Dependency: libnghttp2-httpd24.so.14()(64bit) for package: httpd24-httpd-2.4.27-8.el7.x86_64
---> Package mariadb.x86_64 1:5.5.56-2.el7 will be installed
--> Processing Dependency: mariadb-libs(x86-64) = 1:5.5.56-2.el7 for package: 1:mariadb-5.5.56-2.el7.x86_64
---> Package mariadb-server.x86_64 1:5.5.56-2.el7 will be installed
--> Processing Dependency: perl-DBD-MySQL for package: 1:mariadb-server-5.5.56-2.el7.x86_64
---> Package php55.x86_64 0:2.0-1.el7 will be installed
--> Processing Dependency: php55-runtime for package: php55-2.0-1.el7.x86_64
--> Processing Dependency: php55-php-pear for package: php55-2.0-1.el7.x86_64
--> Processing Dependency: php55-php-common for package: php55-2.0-1.el7.x86_64
--> Processing Dependency: php55-php-cli for package: php55-2.0-1.el7.x86_64
---> Package php55-php.x86_64 0:5.5.21-5.el7 will be installed
---> Package php55-php-gd.x86_64 0:5.5.21-5.el7 will be installed
--> Processing Dependency: libt1.so.5()(64bit) for package: php55-php-gd-5.5.21-5.el7.x86_64
---> Package php55-php-mbstring.x86_64 0:5.5.21-5.el7 will be installed
---> Package php55-php-mysqlnd.x86_64 0:5.5.21-5.el7 will be installed
--> Processing Dependency: php55-php-pdo(x86-64) = 5.5.21-5.el7 for package: php55-php-mysqlnd-5.5.21-5.el7.x86_64
---> Package php55-php-xml.x86_64 0:5.5.21-5.el7 will be installed
--> Running transaction check
---> Package httpd24-httpd-tools.x86_64 0:2.4.27-8.el7 will be installed
---> Package httpd24-libnghttp2.x86_64 0:1.7.1-6.el7 will be installed
---> Package httpd24-runtime.x86_64 0:1.1-18.el7 will be installed
---> Package mariadb-libs.x86_64 1:5.5.52-1.el7 will be updated
---> Package mariadb-libs.x86_64 1:5.5.56-2.el7 will be an update
---> Package perl-DBD-MySQL.x86_64 0:4.023-5.el7 will be installed
---> Package php55-php-cli.x86_64 0:5.5.21-5.el7 will be installed
---> Package php55-php-common.x86_64 0:5.5.21-5.el7 will be installed
--> Processing Dependency: php55-php-pecl-jsonc(x86-64) for package: php55-php-common-5.5.21-5.el7.x86_64
--> Processing Dependency: libzip.so.2()(64bit) for package: php55-php-common-5.5.21-5.el7.x86_64
---> Package php55-php-pdo.x86_64 0:5.5.21-5.el7 will be installed
---> Package php55-php-pear.noarch 1:1.9.4-10.el7 will be installed
--> Processing Dependency: php55-php-posix for package: 1:php55-php-pear-1.9.4-10.el7.noarch
---> Package php55-runtime.x86_64 0:2.0-1.el7 will be installed
---> Package t1lib.x86_64 0:5.1.2-14.el7 will be installed
--> Running transaction check
---> Package libzip.x86_64 0:0.10.1-8.el7 will be installed
---> Package php55-php-pecl-jsonc.x86_64 0:1.3.5-1.el7 will be installed
---> Package php55-php-process.x86_64 0:5.5.21-5.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                 Arch      Version              Repository         Size
================================================================================
Installing:
 httpd24-httpd           x86_64    2.4.27-8.el7         centos-sclo-rh    1.4 M
 mariadb                 x86_64    1:5.5.56-2.el7       base              8.7 M
 mariadb-server          x86_64    1:5.5.56-2.el7       base               11 M
 php55                   x86_64    2.0-1.el7            centos-sclo-rh    3.1 k
 php55-php               x86_64    5.5.21-5.el7         centos-sclo-rh    1.3 M
 php55-php-gd            x86_64    5.5.21-5.el7         centos-sclo-rh    154 k
 php55-php-mbstring      x86_64    5.5.21-5.el7         centos-sclo-rh    516 k
 php55-php-mysqlnd       x86_64    5.5.21-5.el7         centos-sclo-rh    1.2 M
 php55-php-xml           x86_64    5.5.21-5.el7         centos-sclo-rh    157 k
Installing for dependencies:
 httpd24-httpd-tools     x86_64    2.4.27-8.el7         centos-sclo-rh     86 k
 httpd24-libnghttp2      x86_64    1.7.1-6.el7          centos-sclo-rh     61 k
 httpd24-runtime         x86_64    1.1-18.el7           centos-sclo-rh     28 k
 libzip                  x86_64    0.10.1-8.el7         base               48 k
 perl-DBD-MySQL          x86_64    4.023-5.el7          base              140 k
 php55-php-cli           x86_64    5.5.21-5.el7         centos-sclo-rh    2.6 M
 php55-php-common        x86_64    5.5.21-5.el7         centos-sclo-rh    680 k
 php55-php-pdo           x86_64    5.5.21-5.el7         centos-sclo-rh     98 k
 php55-php-pear          noarch    1:1.9.4-10.el7       centos-sclo-rh    357 k
 php55-php-pecl-jsonc    x86_64    1.3.5-1.el7          centos-sclo-rh     40 k
 php55-php-process       x86_64    5.5.21-5.el7         centos-sclo-rh     58 k
 php55-runtime           x86_64    2.0-1.el7            centos-sclo-rh    1.1 M
 t1lib                   x86_64    5.1.2-14.el7         base              166 k
Updating for dependencies:
 mariadb-libs            x86_64    1:5.5.56-2.el7       base              757 k

Transaction Summary
================================================================================
Install  9 Packages (+13 Dependent packages)
Upgrade             (  1 Dependent package)

Total download size: 31 M
Is this ok [y/d/N]: y
Downloading packages:
No Presto metadata available for base
warning: /var/cache/yum/x86_64/7/centos-sclo-rh/packages/httpd24-httpd-tools-2.4.27-8.el7.x86_64.rpm: Header V4 RSA/SHA1 Signature, key ID f2ee9d55: NOKEY
Public key for httpd24-httpd-tools-2.4.27-8.el7.x86_64.rpm is not installed
(1/23): httpd24-httpd-tools-2.4.27-8.el7.x86_64.rpm        |  86 kB   00:01     
(2/23): httpd24-libnghttp2-1.7.1-6.el7.x86_64.rpm          |  61 kB   00:01     
(3/23): libzip-0.10.1-8.el7.x86_64.rpm                     |  48 kB   00:00     
(4/23): mariadb-libs-5.5.56-2.el7.x86_64.rpm               | 757 kB   00:00     
(5/23): perl-DBD-MySQL-4.023-5.el7.x86_64.rpm              | 140 kB   00:00     
(6/23): httpd24-runtime-1.1-18.el7.x86_64.rpm              |  28 kB   00:00     
(7/23): php55-2.0-1.el7.x86_64.rpm                         | 3.1 kB   00:00     
(8/23): httpd24-httpd-2.4.27-8.el7.x86_64.rpm              | 1.4 MB   00:06     
(9/23): php55-php-5.5.21-5.el7.x86_64.rpm                  | 1.3 MB   00:04     
(10/23): mariadb-5.5.56-2.el7.x86_64.rpm                   | 8.7 MB   00:11     
(11/23): php55-php-cli-5.5.21-5.el7.x86_64.rpm             | 2.6 MB   00:09     
(12/23): php55-php-common-5.5.21-5.el7.x86_64.rpm          | 680 kB   00:07     
(13/23): php55-php-gd-5.5.21-5.el7.x86_64.rpm              | 154 kB   00:01     
(14/23): php55-php-mbstring-5.5.21-5.el7.x86_64.rpm        | 516 kB   00:04     
(15/23): php55-php-mysqlnd-5.5.21-5.el7.x86_64.rpm         | 1.2 MB   00:04     
(16/23): php55-php-pdo-5.5.21-5.el7.x86_64.rpm             |  98 kB   00:01     
(17/23): php55-php-pecl-jsonc-1.3.5-1.el7.x86_64.rpm       |  40 kB   00:00     
(18/23): php55-php-process-5.5.21-5.el7.x86_64.rpm         |  58 kB   00:01     
(19/23): php55-php-pear-1.9.4-10.el7.noarch.rpm            | 357 kB   00:03     
(20/23): t1lib-5.1.2-14.el7.x86_64.rpm                     | 166 kB   00:00     
(21/23): php55-php-xml-5.5.21-5.el7.x86_64.rpm             | 157 kB   00:01     
(22/23): mariadb-server-5.5.56-2.el7.x86_64.rpm            |  11 MB   00:23     
(23/23): php55-runtime-2.0-1.el7.x86_64.rpm                | 1.1 MB   00:09     
--------------------------------------------------------------------------------
Total                                              911 kB/s |  31 MB  00:34     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
Importing GPG key 0xF2EE9D55:
 Userid     : "CentOS SoftwareCollections SIG (https://wiki.centos.org/SpecialInterestGroup/SCLo) <security@centos.org>"
 Fingerprint: c4db d535 b1fb ba14 f8ba 64a8 4eb8 4e71 f2ee 9d55
 Package    : centos-release-scl-rh-2-2.el7.centos.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-SIG-SCLo
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Updating   : 1:mariadb-libs-5.5.56-2.el7.x86_64                          1/24 
  Installing : php55-runtime-2.0-1.el7.x86_64                              2/24 
  Installing : httpd24-runtime-1.1-18.el7.x86_64                           3/24 
  Installing : httpd24-libnghttp2-1.7.1-6.el7.x86_64                       4/24 
  Installing : perl-DBD-MySQL-4.023-5.el7.x86_64                           5/24 
  Installing : 1:mariadb-5.5.56-2.el7.x86_64                               6/24 
  Installing : libzip-0.10.1-8.el7.x86_64                                  7/24 
  Installing : php55-php-cli-5.5.21-5.el7.x86_64                           8/24 
  Installing : php55-php-process-5.5.21-5.el7.x86_64                       9/24 
  Installing : php55-php-xml-5.5.21-5.el7.x86_64                          10/24 
  Installing : php55-php-common-5.5.21-5.el7.x86_64                       11/24 
  Installing : 1:php55-php-pear-1.9.4-10.el7.noarch                       12/24 
  Installing : php55-php-pecl-jsonc-1.3.5-1.el7.x86_64                    13/24 
  Installing : php55-php-pdo-5.5.21-5.el7.x86_64                          14/24 
  Installing : t1lib-5.1.2-14.el7.x86_64                                  15/24 
  Installing : httpd24-httpd-tools-2.4.27-8.el7.x86_64                    16/24 
  Installing : httpd24-httpd-2.4.27-8.el7.x86_64                          17/24 
  Installing : php55-php-5.5.21-5.el7.x86_64                              18/24 
  Installing : php55-php-gd-5.5.21-5.el7.x86_64                           19/24 
  Installing : php55-php-mysqlnd-5.5.21-5.el7.x86_64                      20/24 
  Installing : php55-2.0-1.el7.x86_64                                     21/24 
  Installing : php55-php-mbstring-5.5.21-5.el7.x86_64                     22/24 
  Installing : 1:mariadb-server-5.5.56-2.el7.x86_64                       23/24 
  Cleanup    : 1:mariadb-libs-5.5.52-1.el7.x86_64                         24/24 
  Verifying  : 1:php55-php-pear-1.9.4-10.el7.noarch                        1/24 
  Verifying  : php55-php-pecl-jsonc-1.3.5-1.el7.x86_64                     2/24 
  Verifying  : php55-runtime-2.0-1.el7.x86_64                              3/24 
  Verifying  : 1:mariadb-server-5.5.56-2.el7.x86_64                        4/24 
  Verifying  : httpd24-runtime-1.1-18.el7.x86_64                           5/24 
  Verifying  : httpd24-httpd-2.4.27-8.el7.x86_64                           6/24 
  Verifying  : php55-php-mysqlnd-5.5.21-5.el7.x86_64                       7/24 
  Verifying  : php55-php-pdo-5.5.21-5.el7.x86_64                           8/24 
  Verifying  : httpd24-libnghttp2-1.7.1-6.el7.x86_64                       9/24 
  Verifying  : httpd24-httpd-tools-2.4.27-8.el7.x86_64                    10/24 
  Verifying  : perl-DBD-MySQL-4.023-5.el7.x86_64                          11/24 
  Verifying  : php55-php-cli-5.5.21-5.el7.x86_64                          12/24 
  Verifying  : t1lib-5.1.2-14.el7.x86_64                                  13/24 
  Verifying  : php55-php-mbstring-5.5.21-5.el7.x86_64                     14/24 
  Verifying  : php55-2.0-1.el7.x86_64                                     15/24 
  Verifying  : php55-php-process-5.5.21-5.el7.x86_64                      16/24 
  Verifying  : php55-php-common-5.5.21-5.el7.x86_64                       17/24 
  Verifying  : php55-php-xml-5.5.21-5.el7.x86_64                          18/24 
  Verifying  : php55-php-5.5.21-5.el7.x86_64                              19/24 
  Verifying  : libzip-0.10.1-8.el7.x86_64                                 20/24 
  Verifying  : 1:mariadb-libs-5.5.56-2.el7.x86_64                         21/24 
  Verifying  : php55-php-gd-5.5.21-5.el7.x86_64                           22/24 
  Verifying  : 1:mariadb-5.5.56-2.el7.x86_64                              23/24 
  Verifying  : 1:mariadb-libs-5.5.52-1.el7.x86_64                         24/24 

Installed:
  httpd24-httpd.x86_64 0:2.4.27-8.el7                                           
  mariadb.x86_64 1:5.5.56-2.el7                                                 
  mariadb-server.x86_64 1:5.5.56-2.el7                                          
  php55.x86_64 0:2.0-1.el7                                                      
  php55-php.x86_64 0:5.5.21-5.el7                                               
  php55-php-gd.x86_64 0:5.5.21-5.el7                                            
  php55-php-mbstring.x86_64 0:5.5.21-5.el7                                      
  php55-php-mysqlnd.x86_64 0:5.5.21-5.el7                                       
  php55-php-xml.x86_64 0:5.5.21-5.el7                                           

Dependency Installed:
  httpd24-httpd-tools.x86_64 0:2.4.27-8.el7                                     
  httpd24-libnghttp2.x86_64 0:1.7.1-6.el7                                       
  httpd24-runtime.x86_64 0:1.1-18.el7                                           
  libzip.x86_64 0:0.10.1-8.el7                                                  
  perl-DBD-MySQL.x86_64 0:4.023-5.el7                                           
  php55-php-cli.x86_64 0:5.5.21-5.el7                                           
  php55-php-common.x86_64 0:5.5.21-5.el7                                        
  php55-php-pdo.x86_64 0:5.5.21-5.el7                                           
  php55-php-pear.noarch 1:1.9.4-10.el7                                          
  php55-php-pecl-jsonc.x86_64 0:1.3.5-1.el7                                     
  php55-php-process.x86_64 0:5.5.21-5.el7                                       
  php55-runtime.x86_64 0:2.0-1.el7                                              
  t1lib.x86_64 0:5.1.2-14.el7                                                   

Dependency Updated:
  mariadb-libs.x86_64 1:5.5.56-2.el7                                            

Complete!
[root@wiki ~]# 
~~~

## 安装与配置 mysql

~~~
[root@wiki ~]# systemctl start mariadb
[root@wiki ~]# ps faux | grep mysql
root      4132  0.0  0.0 112648   964 pts/0    S+   23:15   0:00                  \_ grep --color=auto mysql
mysql     3933  0.0  0.0 113252  1592 ?        Ss   23:15   0:00 /bin/sh /usr/bin/mysqld_safe --basedir=/usr
mysql     4095  1.5  3.9 903284 81452 ?        Sl   23:15   0:00  \_ /usr/libexec/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --log-error=/var/log/mariadb/mariadb.log --pid-file=/var/run/mariadb/mariadb.pid --socket=/var/lib/mysql/mysql.sock
[root@wiki ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)
Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] Y     
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] Y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] Y
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] Y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] Y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
[root@wiki ~]# 
~~~

## 创建用户与数据库

并且配置合适的权限给这个用户

~~~
[root@wiki ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 5.5.56-MariaDB MariaDB Server

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> CREATE USER 'wiki'@'localhost' IDENTIFIED BY 'wiki';Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> CREATE DATABASE wiki_db;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON wiki_db.* TO 'wiki'@'localhost';
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| wiki_db            |
+--------------------+
4 rows in set (0.00 sec)

MariaDB [(none)]> SHOW GRANTS FOR 'wiki'@'localhost';
+-------------------------------------------------------------------------------------------------------------+
| Grants for wiki@localhost                                                                                   |
+-------------------------------------------------------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'wiki'@'localhost' IDENTIFIED BY PASSWORD '*A5DB2D927D6DF94DA5E1CE4B293AEAAB4D8304EA' |
| GRANT ALL PRIVILEGES ON `wiki_db`.* TO 'wiki'@'localhost'                                                   |
+-------------------------------------------------------------------------------------------------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> exit
Bye
[root@wiki ~]# 
~~~


## 启用服务

~~~
[root@wiki ~]# systemctl enable mariadb
Created symlink from /etc/systemd/system/multi-user.target.wants/mariadb.service to /usr/lib/systemd/system/mariadb.service.
[root@wiki ~]# 
[root@wiki ~]# systemctl enable httpd24-httpd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd24-httpd.service to /usr/lib/systemd/system/httpd24-httpd.service.
[root@wiki ~]# 
~~~

## 下载 mediawiki

~~~
[root@wiki wiki]# pwd
/root/wiki
[root@wiki wiki]# ls
[root@wiki wiki]# wget http://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz
--2018-04-12 23:29:28--  http://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz
Resolving releases.wikimedia.org (releases.wikimedia.org)... 208.80.153.248, 2620:0:860:ed1a::3:d
Connecting to releases.wikimedia.org (releases.wikimedia.org)|208.80.153.248|:80... connected.
HTTP request sent, awaiting response... 301 TLS Redirect
Location: https://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz [following]
--2018-04-12 23:29:29--  https://releases.wikimedia.org/mediawiki/1.30/mediawiki-1.30.0.tar.gz
Connecting to releases.wikimedia.org (releases.wikimedia.org)|208.80.153.248|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 42680041 (41M) [application/x-gzip]
Saving to: ‘mediawiki-1.30.0.tar.gz’

100%[======================================>] 42,680,041  2.21MB/s   in 40s    

2018-04-12 23:30:09 (1.03 MB/s) - ‘mediawiki-1.30.0.tar.gz’ saved [42680041/42680041]

[root@wiki wiki]# 
~~~

## 解压 mediawiki 到合适的路径

~~~
[root@wiki wiki]# cd /var/www/
[root@wiki www]# tar -zxf /root/wiki/mediawiki-1.30.0.tar.gz
[root@wiki www]# ln -s mediawiki-1.30.0/ mediawiki
[root@wiki www]# ll
total 4
drwxr-xr-x.  2 root root     6 Apr 13  2017 cgi-bin
drwxr-xr-x.  2 root root     6 Apr 13  2017 html
lrwxrwxrwx.  1 root root    17 Apr 12 23:34 mediawiki -> mediawiki-1.30.0/
drwxr-xr-x. 15  501 games 4096 Dec  9 07:19 mediawiki-1.30.0
[root@wiki www]# 
~~~

## 调整 Apache 配置

设定根目录路径

设定 index 文件名

~~~
[root@wiki www]# vim /etc/httpd/conf/httpd.conf 
[root@wiki www]# grep DocumentRoot /etc/httpd/conf/httpd.conf
# DocumentRoot: The directory out of which you will serve your
#DocumentRoot "/var/www/html"
DocumentRoot "/var/www"
    # access content that does not live under the DocumentRoot.
[root@wiki www]# grep DirectoryIndex /etc/httpd/conf/httpd.conf
    DirectoryIndex index.html index.html.var index.php
# DirectoryIndex: sets the file that Apache will serve if a directory
    DirectoryIndex index.html
[root@wiki www]# 
~~~

## 启动 Apache 服务

~~~
[root@wiki www]# systemctl status httpd24-httpd.service
● httpd24-httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd24-httpd.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
[root@wiki www]# systemctl start httpd24-httpd.service
[root@wiki www]# systemctl status httpd24-httpd.service
● httpd24-httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd24-httpd.service; enabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-04-12 23:50:09 CST; 2s ago
 Main PID: 5233 (httpd)
   Status: "Processing requests..."
   CGroup: /system.slice/httpd24-httpd.service
           ├─5233 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─5234 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─5235 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─5236 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           ├─5237 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND
           └─5238 /opt/rh/httpd24/root/usr/sbin/httpd -DFOREGROUND

Apr 12 23:50:08 wiki systemd[1]: Starting The Apache HTTP Server...
Apr 12 23:50:09 wiki httpd-scl-wrapper[5233]: AH00558: httpd: Could not reli...e
Apr 12 23:50:09 wiki systemd[1]: Started The Apache HTTP Server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@wiki www]# 
~~~

## 开启防火墙

允许 80 口的访问

~~~
[root@wiki www]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: dhcpv6-client ssh
  ports: 1194/udp 8080/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  sourceports: 
  icmp-blocks: 
  rich rules: 
	
[root@wiki www]# netstat -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:3306            0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN     
tcp        0      0 192.168.122.1:53        0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:631           0.0.0.0:*               LISTEN     
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN     
tcp        0      0 192.168.56.210:22       192.168.56.1:59612      ESTABLISHED
tcp        0      0 192.168.56.210:22       192.168.56.1:57378      ESTABLISHED
tcp6       0      0 :::111                  :::*                    LISTEN     
tcp6       0      0 :::80                   :::*                    LISTEN     
tcp6       0      0 :::22                   :::*                    LISTEN     
tcp6       0      0 ::1:631                 :::*                    LISTEN     
tcp6       0      0 ::1:25                  :::*                    LISTEN     
[root@wiki www]# 
[root@wiki www]# firewall-cmd --add-port 80/tcp --permanent
success
[root@wiki www]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: dhcpv6-client ssh
  ports: 1194/udp 8080/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  sourceports: 
  icmp-blocks: 
  rich rules: 
	
[root@wiki www]# firewall-cmd --reload
success
[root@wiki www]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: dhcpv6-client ssh
  ports: 80/tcp 1194/udp 8080/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  sourceports: 
  icmp-blocks: 
  rich rules: 
	
[root@wiki www]# 
~~~

### 直接在防火墙中添加服务

~~~
[root@wiki www]# firewall-cmd --add-service=http
success
[root@wiki www]# firewall-cmd --add-service=https
success
[root@wiki www]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s3 enp0s8
  sources: 
  services: dhcpv6-client http https ssh
  ports: 80/tcp 1194/udp 8080/tcp
  protocols: 
  masquerade: no
  forward-ports: 
  sourceports: 
  icmp-blocks: 
  rich rules: 
	
[root@wiki www]# 
~~~


## SELINUX

如果开启了 SELINUX

则通过以下方式来调整目录的 context

~~~
[root@wiki www]# getenforce 
Enforcing
[root@wiki www]# ls -lZ /var/www/
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_content_t:s0 html
lrwxrwxrwx. root   root   unconfined_u:object_r:httpd_sys_content_t:s0 mediawiki -> mediawiki-1.30.0/
drwxr-xr-x. apache apache unconfined_u:object_r:httpd_sys_content_t:s0 mediawiki-1.30.0
[root@wiki www]# restorecon -FR /var/www/mediawiki-1.30.0/
[root@wiki www]# ls -lZ /var/www/
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_content_t:s0 html
lrwxrwxrwx. root   root   unconfined_u:object_r:httpd_sys_content_t:s0 mediawiki -> mediawiki-1.30.0/
drwxr-xr-x. apache apache system_u:object_r:httpd_sys_content_t:s0 mediawiki-1.30.0
[root@wiki www]# restorecon -FR /var/www/mediawiki
[root@wiki www]# ls -lZ /var/www/
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
drwxr-xr-x. root   root   system_u:object_r:httpd_sys_content_t:s0 html
lrwxrwxrwx. root   root   system_u:object_r:httpd_sys_content_t:s0 mediawiki -> mediawiki-1.30.0/
drwxr-xr-x. apache apache system_u:object_r:httpd_sys_content_t:s0 mediawiki-1.30.0
[root@wiki www]# 
~~~










其实这条默认路由也可以由 server 端直接推送过来

其实，这条路由朝外，就是在穿透放火墙访问外网资源

如果这条路由朝内，就是在穿透放火墙访问内网资源，这两者并无本质区别

致此，已经完成了 linux 下 openvpn 客户端的构建与配置


---

# 总结


客户端的配置相对简单，在服务端管理好转发是关键


* TOC
{:toc}


---



[mediawiki]:https://www.mediawiki.org/wiki/MediaWiki
[mediawiki_install]:https://www.mediawiki.org/wiki/Manual:Running_MediaWiki_on_Red_Hat_Linux