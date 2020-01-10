---
layout: post
title: "Install SonarQube"
date: 2020-01-10 06:33:57
image: '/assets/img/'
description: 'SonarQube 的安装方法'
main-class: sonarqube
color: 
tags:
 - sonarqube
 - postgresql
 - tools
 - java
categories:
 - sonarqube
twitter_text: 'simple process of sonarqube installation'
introduction: 'installation method of sonarqube'
---

# 前言


**[SonarQube][sonarqube]** 是一个很受欢迎的静态代码扫描软件

>Your teammate for Code Quality and Security
>SonarQube empowers all developers to write cleaner and safer code

能非常方便地集成到 CI/CD 环境中

主要具备以下特性

~~~
● 目前可以支持27种语言
● 可以侦测 Bug 和 缺陷
● 可以 review 安全热点
● 可以跟踪代码风味和修复技术债
● 代码质量评价和历史
● 可以扩展插件 60+
~~~


同类软件有:

* pmd
* checkstyle
* findbugs
* Jtest

区别为:

* pmd: 基于源代码分析, 主要面向安全编码规则, 如 "避免声明同名变量", 包括风格类、类型使用等等, 具备一定的数据流分析和路径分析能力
* checkstyle: 基于源代码, 与pmd类似, 但更侧重编码的语法风格, 分析深度不及pmd
* findbugs: 基于字节码分析, 大量使用数据流分析技术, 侧重运行时错误检测, 如空指针引用等, 分析深度大于前述两个
* sonar: 定位是代码质量平台, 本身不进行代码分析, 但可以集成各个静态分析工具以及其他软件开发测试工具, 并基于集成工具的结果数据按照一定的质量模型, 如iso-9126, 对软件的质量进行评估
* Jtest: 能够按照其内置的超过 800 条的 Java 编码规范自动检查并纠正这些隐蔽且难以修复的编码错误, 同时, 还支持用户自定义编码规则, 帮助用户预防一些特殊用法的错误, 着重于发现代码缺陷

参考 **[常用 Java 静态代码分析工具的分析与比较][statictest]**

在开发任务繁重的情况下, 使用静态代码检测工具来协助检查，能显著减少在代码逐行检查上花费的时间，提高软件可靠性并节省软件开发和测试成本

这里分享一下 **[SonarQube][sonarqube]** server 的安装方法

参考 **[Install the Server][sonarqube_install]**

> **Tip:** 当前版本 **SonarQube Version: 8.1** Release on December 2019


## 依赖

### Java

* OpenJDK 11 

### Hardware Requirements

* \>= 3G RAM (2G for Software 1G for OS)
* 64bit 

### Enterprise Hardware Recommendations

* \>= 16G RAM
* 8 cores CPU
* 64bit

### 数据库

* PostgreSQL 10 9.3-9.6
* Microsoft SQL Server 2014 2016 2017
* Oracle  XE Editions 11G 12C 18C 19C

### OS

因为它会使用到 ElasticSearch 所以针对 Linux 的系统限制有如下要求

* vm.max_map_count is greater or equals to 262144
* fs.file-max is greater or equals to 65536
* the user running SonarQube can open at least 65536 file descriptors
* the user running SonarQube can open at least 4096 threads

参考 **[Prerequisites and Overview][requirements]**

---



# 操作

## 系统环境

~~~
[root@sonarqube ~]# hostnamectl
   Static hostname: sonarqube
         Icon name: computer-vm
           Chassis: vm
        Machine ID: a05923a33b584c9ba7d8247889f09194
           Boot ID: a5c55b1133164fb3b1b72062ca5e7714
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@sonarqube ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 81282sec preferred_lft 81282sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:6b:b4:f8 brd ff:ff:ff:ff:ff:ff
    inet 12.0.0.13/24 brd 12.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe6b:b4f8/64 scope link
       valid_lft forever preferred_lft forever
[root@sonarqube ~]#
~~~

## 安装 java

~~~
[root@sonarqube ~]# yum -y install java-11-openjdk
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.hostlink.com.hk
 * extras: mirror.hostlink.com.hk
 * updates: mirror.hostlink.com.hk
Resolving Dependencies
--> Running transaction check
---> Package java-11-openjdk.x86_64 1:11.0.5.10-0.el7_7 will be installed
--> Processing Dependency: java-11-openjdk-headless(x86-64) = 1:11.0.5.10-0.el7_7 for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: xorg-x11-fonts-Type1 for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libjpeg.so.62(LIBJPEG_6.2)(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: fontconfig(x86-64) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libjvm.so()(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libjpeg.so.62()(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libjava.so()(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libgif.so.4()(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libXtst.so.6()(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libXrender.so.1()(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libXi.so.6()(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libXext.so.6()(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libX11.so.6()(64bit) for package: 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64
--> Running transaction check
---> Package fontconfig.x86_64 0:2.13.0-4.3.el7 will be installed
--> Processing Dependency: fontpackages-filesystem for package: fontconfig-2.13.0-4.3.el7.x86_64
--> Processing Dependency: dejavu-sans-fonts for package: fontconfig-2.13.0-4.3.el7.x86_64
---> Package giflib.x86_64 0:4.1.6-9.el7 will be installed
--> Processing Dependency: libSM.so.6()(64bit) for package: giflib-4.1.6-9.el7.x86_64
--> Processing Dependency: libICE.so.6()(64bit) for package: giflib-4.1.6-9.el7.x86_64
---> Package java-11-openjdk-headless.x86_64 1:11.0.5.10-0.el7_7 will be installed
--> Processing Dependency: tzdata-java >= 2015d for package: 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: copy-jdk-configs >= 3.3 for package: 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: pcsc-lite-libs(x86-64) for package: 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: lksctp-tools(x86-64) for package: 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libasound.so.2(ALSA_0.9.0rc4)(64bit) for package: 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libasound.so.2(ALSA_0.9)(64bit) for package: 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: javapackages-tools for package: 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64
--> Processing Dependency: libasound.so.2()(64bit) for package: 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64
---> Package libX11.x86_64 0:1.6.7-2.el7 will be installed
--> Processing Dependency: libX11-common >= 1.6.7-2.el7 for package: libX11-1.6.7-2.el7.x86_64
--> Processing Dependency: libxcb.so.1()(64bit) for package: libX11-1.6.7-2.el7.x86_64
---> Package libXext.x86_64 0:1.3.3-3.el7 will be installed
---> Package libXi.x86_64 0:1.7.9-1.el7 will be installed
---> Package libXrender.x86_64 0:0.9.10-1.el7 will be installed
---> Package libXtst.x86_64 0:1.2.3-1.el7 will be installed
---> Package libjpeg-turbo.x86_64 0:1.2.90-8.el7 will be installed
---> Package xorg-x11-fonts-Type1.noarch 0:7.5-9.el7 will be installed
--> Processing Dependency: ttmkfdir for package: xorg-x11-fonts-Type1-7.5-9.el7.noarch
--> Processing Dependency: ttmkfdir for package: xorg-x11-fonts-Type1-7.5-9.el7.noarch
--> Processing Dependency: mkfontdir for package: xorg-x11-fonts-Type1-7.5-9.el7.noarch
--> Processing Dependency: mkfontdir for package: xorg-x11-fonts-Type1-7.5-9.el7.noarch
--> Running transaction check
---> Package alsa-lib.x86_64 0:1.1.8-1.el7 will be installed
---> Package copy-jdk-configs.noarch 0:3.3-10.el7_5 will be installed
---> Package dejavu-sans-fonts.noarch 0:2.33-6.el7 will be installed
--> Processing Dependency: dejavu-fonts-common = 2.33-6.el7 for package: dejavu-sans-fonts-2.33-6.el7.noarch
---> Package fontpackages-filesystem.noarch 0:1.44-8.el7 will be installed
---> Package javapackages-tools.noarch 0:3.4.1-11.el7 will be installed
--> Processing Dependency: python-javapackages = 3.4.1-11.el7 for package: javapackages-tools-3.4.1-11.el7.noarch
---> Package libICE.x86_64 0:1.0.9-9.el7 will be installed
---> Package libSM.x86_64 0:1.2.2-2.el7 will be installed
---> Package libX11-common.noarch 0:1.6.7-2.el7 will be installed
---> Package libxcb.x86_64 0:1.13-1.el7 will be installed
--> Processing Dependency: libXau.so.6()(64bit) for package: libxcb-1.13-1.el7.x86_64
---> Package lksctp-tools.x86_64 0:1.0.17-2.el7 will be installed
---> Package pcsc-lite-libs.x86_64 0:1.8.8-8.el7 will be installed
---> Package ttmkfdir.x86_64 0:3.0.9-42.el7 will be installed
---> Package tzdata-java.noarch 0:2019c-1.el7 will be installed
---> Package xorg-x11-font-utils.x86_64 1:7.5-21.el7 will be installed
--> Processing Dependency: libfontenc.so.1()(64bit) for package: 1:xorg-x11-font-utils-7.5-21.el7.x86_64
--> Running transaction check
---> Package dejavu-fonts-common.noarch 0:2.33-6.el7 will be installed
---> Package libXau.x86_64 0:1.0.8-2.1.el7 will be installed
---> Package libfontenc.x86_64 0:1.1.3-3.el7 will be installed
---> Package python-javapackages.noarch 0:3.4.1-11.el7 will be installed
--> Processing Dependency: python-lxml for package: python-javapackages-3.4.1-11.el7.noarch
--> Running transaction check
---> Package python-lxml.x86_64 0:3.2.1-4.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==================================================================================
 Package                     Arch      Version                   Repository  Size
==================================================================================
Installing:
 java-11-openjdk             x86_64    1:11.0.5.10-0.el7_7       updates    212 k
Installing for dependencies:
 alsa-lib                    x86_64    1.1.8-1.el7               base       425 k
 copy-jdk-configs            noarch    3.3-10.el7_5              base        21 k
 dejavu-fonts-common         noarch    2.33-6.el7                base        64 k
 dejavu-sans-fonts           noarch    2.33-6.el7                base       1.4 M
 fontconfig                  x86_64    2.13.0-4.3.el7            base       254 k
 fontpackages-filesystem     noarch    1.44-8.el7                base       9.9 k
 giflib                      x86_64    4.1.6-9.el7               base        40 k
 java-11-openjdk-headless    x86_64    1:11.0.5.10-0.el7_7       updates     39 M
 javapackages-tools          noarch    3.4.1-11.el7              base        73 k
 libICE                      x86_64    1.0.9-9.el7               base        66 k
 libSM                       x86_64    1.2.2-2.el7               base        39 k
 libX11                      x86_64    1.6.7-2.el7               base       607 k
 libX11-common               noarch    1.6.7-2.el7               base       164 k
 libXau                      x86_64    1.0.8-2.1.el7             base        29 k
 libXext                     x86_64    1.3.3-3.el7               base        39 k
 libXi                       x86_64    1.7.9-1.el7               base        40 k
 libXrender                  x86_64    0.9.10-1.el7              base        26 k
 libXtst                     x86_64    1.2.3-1.el7               base        20 k
 libfontenc                  x86_64    1.1.3-3.el7               base        31 k
 libjpeg-turbo               x86_64    1.2.90-8.el7              base       135 k
 libxcb                      x86_64    1.13-1.el7                base       214 k
 lksctp-tools                x86_64    1.0.17-2.el7              base        88 k
 pcsc-lite-libs              x86_64    1.8.8-8.el7               base        34 k
 python-javapackages         noarch    3.4.1-11.el7              base        31 k
 python-lxml                 x86_64    3.2.1-4.el7               base       758 k
 ttmkfdir                    x86_64    3.0.9-42.el7              base        48 k
 tzdata-java                 noarch    2019c-1.el7               updates    187 k
 xorg-x11-font-utils         x86_64    1:7.5-21.el7              base       104 k
 xorg-x11-fonts-Type1        noarch    7.5-9.el7                 base       521 k

Transaction Summary
==================================================================================
Install  1 Package (+29 Dependent packages)

Total download size: 44 M
Installed size: 181 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/base/packages/copy-jdk-configs-3.3-10.el7_5.noarch.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for copy-jdk-configs-3.3-10.el7_5.noarch.rpm is not installed
(1/30): copy-jdk-configs-3.3-10.el7_5.noarch.rpm           |  21 kB  00:00:00
(2/30): fontpackages-filesystem-1.44-8.el7.noarch.rpm      | 9.9 kB  00:00:00
(3/30): giflib-4.1.6-9.el7.x86_64.rpm                      |  40 kB  00:00:00
(4/30): alsa-lib-1.1.8-1.el7.x86_64.rpm                    | 425 kB  00:00:01
(5/30): dejavu-fonts-common-2.33-6.el7.noarch.rpm          |  64 kB  00:00:01
(6/30): fontconfig-2.13.0-4.3.el7.x86_64.rpm               | 254 kB  00:00:02
(7/30): dejavu-sans-fonts-2.33-6.el7.noarch.rpm            | 1.4 MB  00:00:03
(8/30): libSM-1.2.2-2.el7.x86_64.rpm                       |  39 kB  00:00:00
(9/30): libICE-1.0.9-9.el7.x86_64.rpm                      |  66 kB  00:00:01
(10/30): libX11-common-1.6.7-2.el7.noarch.rpm              | 164 kB  00:00:00
(11/30): javapackages-tools-3.4.1-11.el7.noarch.rpm        |  73 kB  00:00:04
(12/30): libXau-1.0.8-2.1.el7.x86_64.rpm                   |  29 kB  00:00:01
(13/30): libXi-1.7.9-1.el7.x86_64.rpm                      |  40 kB  00:00:00
(14/30): libXrender-0.9.10-1.el7.x86_64.rpm                |  26 kB  00:00:00
(15/30): libXtst-1.2.3-1.el7.x86_64.rpm                    |  20 kB  00:00:00
Public key for java-11-openjdk-11.0.5.10-0.el7_7.x86_64.rpm is not installed
(16/30): java-11-openjdk-11.0.5.10-0.el7_7.x86_64.rpm      | 212 kB  00:00:05
(17/30): libXext-1.3.3-3.el7.x86_64.rpm                    |  39 kB  00:00:00
(18/30): libfontenc-1.1.3-3.el7.x86_64.rpm                 |  31 kB  00:00:00
(19/30): lksctp-tools-1.0.17-2.el7.x86_64.rpm              |  88 kB  00:00:00
(20/30): pcsc-lite-libs-1.8.8-8.el7.x86_64.rpm             |  34 kB  00:00:00
(21/30): python-javapackages-3.4.1-11.el7.noarch.rpm       |  31 kB  00:00:00
(22/30): libjpeg-turbo-1.2.90-8.el7.x86_64.rpm             | 135 kB  00:00:00
(23/30): ttmkfdir-3.0.9-42.el7.x86_64.rpm                  |  48 kB  00:00:00
(24/30): python-lxml-3.2.1-4.el7.x86_64.rpm                | 758 kB  00:00:02
(25/30): libxcb-1.13-1.el7.x86_64.rpm                      | 214 kB  00:00:03
(26/30): xorg-x11-font-utils-7.5-21.el7.x86_64.rpm         | 104 kB  00:00:00
(27/30): tzdata-java-2019c-1.el7.noarch.rpm                | 187 kB  00:00:03
(28/30): libX11-1.6.7-2.el7.x86_64.rpm                     | 607 kB  00:00:20
(29/30): xorg-x11-fonts-Type1-7.5-9.el7.noarch.rpm         | 521 kB  00:00:15
(30/30): java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64 |  39 MB  00:01:06
----------------------------------------------------------------------------------
Total                                                672 kB/s |  44 MB  01:07
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-6.1810.2.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libjpeg-turbo-1.2.90-8.el7.x86_64                             1/30
  Installing : fontpackages-filesystem-1.44-8.el7.noarch                     2/30
  Installing : libICE-1.0.9-9.el7.x86_64                                     3/30
  Installing : libSM-1.2.2-2.el7.x86_64                                      4/30
  Installing : dejavu-fonts-common-2.33-6.el7.noarch                         5/30
  Installing : dejavu-sans-fonts-2.33-6.el7.noarch                           6/30
  Installing : fontconfig-2.13.0-4.3.el7.x86_64                              7/30
  Installing : libfontenc-1.1.3-3.el7.x86_64                                 8/30
  Installing : 1:xorg-x11-font-utils-7.5-21.el7.x86_64                       9/30
  Installing : libXau-1.0.8-2.1.el7.x86_64                                  10/30
  Installing : libxcb-1.13-1.el7.x86_64                                     11/30
  Installing : pcsc-lite-libs-1.8.8-8.el7.x86_64                            12/30
  Installing : lksctp-tools-1.0.17-2.el7.x86_64                             13/30
  Installing : tzdata-java-2019c-1.el7.noarch                               14/30
  Installing : libX11-common-1.6.7-2.el7.noarch                             15/30
  Installing : libX11-1.6.7-2.el7.x86_64                                    16/30
  Installing : libXext-1.3.3-3.el7.x86_64                                   17/30
  Installing : libXi-1.7.9-1.el7.x86_64                                     18/30
  Installing : libXtst-1.2.3-1.el7.x86_64                                   19/30
  Installing : giflib-4.1.6-9.el7.x86_64                                    20/30
  Installing : libXrender-0.9.10-1.el7.x86_64                               21/30
  Installing : copy-jdk-configs-3.3-10.el7_5.noarch                         22/30
  Installing : alsa-lib-1.1.8-1.el7.x86_64                                  23/30
  Installing : ttmkfdir-3.0.9-42.el7.x86_64                                 24/30
  Installing : xorg-x11-fonts-Type1-7.5-9.el7.noarch                        25/30
  Installing : python-lxml-3.2.1-4.el7.x86_64                               26/30
  Installing : python-javapackages-3.4.1-11.el7.noarch                      27/30
  Installing : javapackages-tools-3.4.1-11.el7.noarch                       28/30
  Installing : 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64          29/30
  Installing : 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64                   30/30
  Verifying  : libXext-1.3.3-3.el7.x86_64                                    1/30
  Verifying  : libXi-1.7.9-1.el7.x86_64                                      2/30
  Verifying  : 1:java-11-openjdk-headless-11.0.5.10-0.el7_7.x86_64           3/30
  Verifying  : fontconfig-2.13.0-4.3.el7.x86_64                              4/30
  Verifying  : giflib-4.1.6-9.el7.x86_64                                     5/30
  Verifying  : libXrender-0.9.10-1.el7.x86_64                                6/30
  Verifying  : 1:xorg-x11-font-utils-7.5-21.el7.x86_64                       7/30
  Verifying  : python-lxml-3.2.1-4.el7.x86_64                                8/30
  Verifying  : libICE-1.0.9-9.el7.x86_64                                     9/30
  Verifying  : fontpackages-filesystem-1.44-8.el7.noarch                    10/30
  Verifying  : ttmkfdir-3.0.9-42.el7.x86_64                                 11/30
  Verifying  : alsa-lib-1.1.8-1.el7.x86_64                                  12/30
  Verifying  : copy-jdk-configs-3.3-10.el7_5.noarch                         13/30
  Verifying  : python-javapackages-3.4.1-11.el7.noarch                      14/30
  Verifying  : dejavu-fonts-common-2.33-6.el7.noarch                        15/30
  Verifying  : libXtst-1.2.3-1.el7.x86_64                                   16/30
  Verifying  : libX11-1.6.7-2.el7.x86_64                                    17/30
  Verifying  : libX11-common-1.6.7-2.el7.noarch                             18/30
  Verifying  : libxcb-1.13-1.el7.x86_64                                     19/30
  Verifying  : tzdata-java-2019c-1.el7.noarch                               20/30
  Verifying  : lksctp-tools-1.0.17-2.el7.x86_64                             21/30
  Verifying  : libjpeg-turbo-1.2.90-8.el7.x86_64                            22/30
  Verifying  : xorg-x11-fonts-Type1-7.5-9.el7.noarch                        23/30
  Verifying  : dejavu-sans-fonts-2.33-6.el7.noarch                          24/30
  Verifying  : pcsc-lite-libs-1.8.8-8.el7.x86_64                            25/30
  Verifying  : javapackages-tools-3.4.1-11.el7.noarch                       26/30
  Verifying  : 1:java-11-openjdk-11.0.5.10-0.el7_7.x86_64                   27/30
  Verifying  : libXau-1.0.8-2.1.el7.x86_64                                  28/30
  Verifying  : libSM-1.2.2-2.el7.x86_64                                     29/30
  Verifying  : libfontenc-1.1.3-3.el7.x86_64                                30/30

Installed:
  java-11-openjdk.x86_64 1:11.0.5.10-0.el7_7

Dependency Installed:
  alsa-lib.x86_64 0:1.1.8-1.el7
  copy-jdk-configs.noarch 0:3.3-10.el7_5
  dejavu-fonts-common.noarch 0:2.33-6.el7
  dejavu-sans-fonts.noarch 0:2.33-6.el7
  fontconfig.x86_64 0:2.13.0-4.3.el7
  fontpackages-filesystem.noarch 0:1.44-8.el7
  giflib.x86_64 0:4.1.6-9.el7
  java-11-openjdk-headless.x86_64 1:11.0.5.10-0.el7_7
  javapackages-tools.noarch 0:3.4.1-11.el7
  libICE.x86_64 0:1.0.9-9.el7
  libSM.x86_64 0:1.2.2-2.el7
  libX11.x86_64 0:1.6.7-2.el7
  libX11-common.noarch 0:1.6.7-2.el7
  libXau.x86_64 0:1.0.8-2.1.el7
  libXext.x86_64 0:1.3.3-3.el7
  libXi.x86_64 0:1.7.9-1.el7
  libXrender.x86_64 0:0.9.10-1.el7
  libXtst.x86_64 0:1.2.3-1.el7
  libfontenc.x86_64 0:1.1.3-3.el7
  libjpeg-turbo.x86_64 0:1.2.90-8.el7
  libxcb.x86_64 0:1.13-1.el7
  lksctp-tools.x86_64 0:1.0.17-2.el7
  pcsc-lite-libs.x86_64 0:1.8.8-8.el7
  python-javapackages.noarch 0:3.4.1-11.el7
  python-lxml.x86_64 0:3.2.1-4.el7
  ttmkfdir.x86_64 0:3.0.9-42.el7
  tzdata-java.noarch 0:2019c-1.el7
  xorg-x11-font-utils.x86_64 1:7.5-21.el7
  xorg-x11-fonts-Type1.noarch 0:7.5-9.el7

Complete!
[root@sonarqube ~]#
[root@sonarqube ~]# java -version
openjdk version "11.0.5" 2019-10-15 LTS
OpenJDK Runtime Environment 18.9 (build 11.0.5+10-LTS)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.5+10-LTS, mixed mode, sharing)
[root@sonarqube ~]#
~~~


## 配置 postgresql


### 添加用户和数据库

~~~
[vagrant@postgresql ~]$ systemctl | grep -i post
  postfix.service                                                                          loaded active running   Postfix Mail Transport Agent
  postgresql-12.service                                                                    loaded active running   PostgreSQL 12 database server
[vagrant@postgresql ~]$
[vagrant@postgresql ~]$ netstat  -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:5432          0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
tcp        0      0 10.0.2.15:22            10.0.2.2:49202          ESTABLISHED
tcp6       0      0 :::111                  :::*                    LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 ::1:5432                :::*                    LISTEN
tcp6       0      0 ::1:25                  :::*                    LISTEN
[vagrant@postgresql ~]$ sudo su - postgres
Last login: Thu Jan  9 09:53:52 UTC 2020 on pts/0
-bash-4.2$ psql
psql (12.1)
Type "help" for help.

postgres=# CREATE USER sonar ;
CREATE ROLE
postgres=# ALTER USER sonar WITH PASSWORD 'sonarqube';
ALTER ROLE
postgres=# CREATE DATABASE sonardb WITH ENCODING 'UTF8';
CREATE DATABASE
postgres=# ALTER DATABASE sonardb OWNER TO sonar;
ALTER DATABASE
postgres=# ALTER USER sonar SET search_path TO sonarqube,public;
ALTER ROLE
postgres=#
postgres=# \q
-bash-4.2$
~~~


### 放开网络连接

~~~
[root@postgresql data]# grep -v "#" pg_hba.conf




local   all             all                                     peer
host    all             all             127.0.0.1/32            ident
host    all             all             ::1/128                 ident
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
[root@postgresql data]# vim pg_hba.conf
[root@postgresql data]# grep -v "#" pg_hba.conf




local   all             all                                     peer
host    all             all             127.0.0.1/32            ident
host    all             all             10.0.0.13/24             md5
host    all             all             ::1/128                 ident
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
[root@postgresql data]#
~~~

### 调整监听的IP

~~~
[root@postgresql data]# grep 'listen_addresses' postgresql.conf
#listen_addresses = 'localhost'		# what IP address(es) to listen on;
[root@postgresql data]# vim postgresql.conf
[root@postgresql data]# grep 'listen_addresses' postgresql.conf
#listen_addresses = 'localhost'		# what IP address(es) to listen on;
listen_addresses = '10.0.0.10'		# what IP address(es) to listen on;
[root@postgresql data]#
~~~

再看看监听的 IP

~~~
[root@postgresql data]# systemctl  restart postgresql-12.service
[root@postgresql data]# netstat  -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 10.0.0.10:5432          0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
tcp        0      0 10.0.2.15:22            10.0.2.2:49202          ESTABLISHED
tcp6       0      0 :::111                  :::*                    LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 ::1:25                  :::*                    LISTEN
[root@postgresql data]#
~~~


## 创建一个 sonar 用户

在 Unix-based 系统中， SonqrQube 不允许以 root 的身份运行，所以要为 sonarqube 创建一个指定的用户

~~~
[root@sonarqube ~]# useradd  sonar
[root@sonarqube ~]# passwd sonar
Changing password for user sonar.
New password:
BAD PASSWORD: The password is shorter than 8 characters
Retype new password:
passwd: all authentication tokens updated successfully.
[root@sonarqube ~]# tail /etc/passwd
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
nfsnobody:x:65534:65534:Anonymous NFS User:/var/lib/nfs:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:995::/var/lib/chrony:/sbin/nologin
vagrant:x:1000:1000:vagrant:/home/vagrant:/bin/bash
sonar:x:1001:1001::/home/sonar:/bin/bash
[root@sonarqube ~]#
~~~

## 创建 schema


测试一下到 postgresql 的连接

~~~
[sonar@sonarqube logs]$ psql -h 10.0.0.10 -U sonar -d sonardb
Password for user sonar:
psql (12.1)
Type "help" for help.

sonardb=> \d
Did not find any relations.
sonardb=> \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
-----------+----------+----------+-------------+-------------+-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 sonardb   | sonar    | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)

sonardb=> \q
[sonar@sonarqube logs]$
~~~

>**Note:** 要提前安装 **postgresql12** 客户端 `yum install postgresql12`

~~~
-bash-4.2$ psql -h 10.0.0.10 -U sonar  -d sonardb
Password for user sonar:
psql (12.1)
Type "help" for help.

sonardb=> \dn
   List of schemas
   Name    |  Owner
-----------+----------
 public    | postgres
(1 rows)

sonardb=> show search_path;
    search_path
-------------------
 sonarqube, public
(1 row)

sonardb=> create schema sonarqube;
CREATE SCHEMA
sonardb=> \dn
   List of schemas
   Name    |  Owner
-----------+----------
 public    | postgres
 sonarqube | sonar
(2 rows)

sonardb=>
~~~

## 下载 sonarqube 的 zip 包

~~~
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.1.0.31237.zip
~~~

~~~
[sonar@sonarqube ~]$ ll
total 206004
-rw-r--r--. 1 sonar sonar 210947038 Jan 10 08:40 sonarqube-8.1.0.31237.zip
[sonar@sonarqube ~]$
~~~

## 解压

~~~
[sonar@sonarqube ~]$ unzip sonarqube-8.1.0.31237.zip
...
...
  inflating: sonarqube-8.1.0.31237/lib/sonar-shutdowner-8.1.0.31237.jar
   creating: sonarqube-8.1.0.31237/elasticsearch/plugins/
[sonar@sonarqube ~]$
~~~

## 配置 sonarqube 数据库连接

~~~
[root@sonarqube sonarqube-8.1.0.31237]# grep -v "#" conf/sonar.properties  | cat -s

[root@sonarqube sonarqube-8.1.0.31237]# vim conf/sonar.properties
[root@sonarqube sonarqube-8.1.0.31237]# grep -v "#" conf/sonar.properties  | cat -s

sonar.jdbc.username=sonar
sonar.jdbc.password=sonarqube

sonar.jdbc.url=jdbc:postgresql://10.0.0.10/sonardb?currentSchema=sonarqube

[root@sonarqube sonarqube-8.1.0.31237]#
~~~

默认情况下除了 Oracle 以外，其它的数据库连接驱动已经安装好了，不要轻易替换，如果是 Oracle, 要将 JDBC driver 拷贝到 `$SONARQUBE-HOME/extesions/jdbc-driver/oracle` 中.

我们用的 postgresql, 不用动

~~~
[root@sonarqube sonarqube-8.1.0.31237]# ll extensions/
total 4
drwxr-xr-x. 3 sonar sonar   20 Dec 17 12:30 jdbc-driver
drwxr-xr-x. 2 sonar sonar 4096 Dec 17 12:30 plugins
[root@sonarqube sonarqube-8.1.0.31237]#
~~~


## 配置 Elasticsearch 的存储路径

默认情况下 Elasticsearch 的数据存储在 `$SONARQUBE-HOME/data` 中, 但是在生产环境中不推荐这么做，建议放到 I/O 更快的位置，可以通过以下参数进行配置 

调整 `$SONARQUBE-HOME/conf/sonar.properties` 中的下面参数 

~~~
# Paths to persistent data files (embedded database and search index) and temporary files.
# Can be absolute or relative to installation directory.
# Defaults are respectively <installation home>/data and <installation home>/temp
#sonar.path.data=data
#sonar.path.temp=temp
~~~

但是在我的实验环境，先不用调整

## 配置 web 服务器

~~~
[root@sonarqube sonarqube-8.1.0.31237]# grep 'sonar.web'  conf/sonar.properties
# enabled with default sizes (50, see property sonar.web.http.maxThreads)
#sonar.web.javaOpts=-Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError
#sonar.web.javaAdditionalOpts=
#sonar.web.host=0.0.0.0
#sonar.web.context=
#sonar.web.port=9000
# based on the sonar.web.connections.acceptCount property. The default value is 50.
#sonar.web.http.maxThreads=50
#sonar.web.http.minThreads=5
#sonar.web.http.acceptCount=25
#sonar.web.sessionTimeoutInMinutes=4320
#sonar.web.systemPasscode=
#sonar.web.sso.enable=false
#sonar.web.sso.loginHeader=X-Forwarded-Login
#sonar.web.sso.nameHeader=X-Forwarded-Name
#sonar.web.sso.emailHeader=X-Forwarded-Email
#sonar.web.sso.groupsHeader=X-Forwarded-Groups
#sonar.web.sso.refreshIntervalInMinutes=5
#sonar.web.accessLogs.enable=true
# Format of access log. It is ignored if sonar.web.accessLogs.enable=false. Possible values are:
#sonar.web.accessLogs.pattern=%i{X-Forwarded-For} %l %u [%t] "%r" %s %b "%i{Referer}" "%i{User-Agent}" "%reqAttribute{ID}"
#sonar.web.accessLogs.pattern=%h %l %u [%t] "%r" %s %b "%i{Referer}" "%i{User-Agent}" "%reqAttribute{ID}"
[root@sonarqube sonarqube-8.1.0.31237]# vim conf/sonar.properties
[root@sonarqube sonarqube-8.1.0.31237]# grep 'sonar.web'  conf/sonar.properties
# enabled with default sizes (50, see property sonar.web.http.maxThreads)
#sonar.web.javaOpts=-Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError
#sonar.web.javaAdditionalOpts=
#sonar.web.host=0.0.0.0
sonar.web.host=0.0.0.0
#sonar.web.context=
sonar.web.context=/sonarqube
#sonar.web.port=9000
sonar.web.port=8000
# based on the sonar.web.connections.acceptCount property. The default value is 50.
#sonar.web.http.maxThreads=50
#sonar.web.http.minThreads=5
#sonar.web.http.acceptCount=25
#sonar.web.sessionTimeoutInMinutes=4320
#sonar.web.systemPasscode=
#sonar.web.sso.enable=false
#sonar.web.sso.loginHeader=X-Forwarded-Login
#sonar.web.sso.nameHeader=X-Forwarded-Name
#sonar.web.sso.emailHeader=X-Forwarded-Email
#sonar.web.sso.groupsHeader=X-Forwarded-Groups
#sonar.web.sso.refreshIntervalInMinutes=5
#sonar.web.accessLogs.enable=true
# Format of access log. It is ignored if sonar.web.accessLogs.enable=false. Possible values are:
#sonar.web.accessLogs.pattern=%i{X-Forwarded-For} %l %u [%t] "%r" %s %b "%i{Referer}" "%i{User-Agent}" "%reqAttribute{ID}"
#sonar.web.accessLogs.pattern=%h %l %u [%t] "%r" %s %b "%i{Referer}" "%i{User-Agent}" "%reqAttribute{ID}"
[root@sonarqube sonarqube-8.1.0.31237]#
~~~

主要调整一下这几个参数

~~~
sonar.web.host=x.x.x.x
sonar.web.port=xxxx
sonar.web.context=/sonarqube
~~~

>**Note:** 因为要非 root 的身份来启动 sonar, 所以这个 port 不用使用 80, 否则 web 无法启动

## 调整系统参数

~~~
[sonar@sonarqube logs]$ sysctl vm.max_map_count
vm.max_map_count = 65530
[sonar@sonarqube logs]$ sysctl fs.file-max
fs.file-max = 286427
[sonar@sonarqube logs]$ ulimit -n
1024
[sonar@sonarqube logs]$ ulimit -u
4096
[sonar@sonarqube logs]$
~~~

修改配置文件

~~~
[root@sonarqube sonarqube-8.1.0.31237]# vim /etc/sysctl.d/99-sonarqube.conf
[root@sonarqube sonarqube-8.1.0.31237]# cat  /etc/sysctl.d/99-sonarqube.conf
vm.max_map_count=262144
fs.file-max=65536
[root@sonarqube sonarqube-8.1.0.31237]# 
[root@sonarqube sonarqube-8.1.0.31237]# vim /etc/security/limits.d/99-sonarqube.conf
[root@sonarqube sonarqube-8.1.0.31237]# cat /etc/security/limits.d/99-sonarqube.conf
sonar   -   nofile   65536
sonar   -   nproc    4096
[root@sonarqube sonarqube-8.1.0.31237]#
~~~

在线(不重启服务器)动态调整

可以使用 root 的身份在当前会话中执行下列的命令直接生效

~~~
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096
~~~


## 运行 sonarqube

~~~
[sonar@sonarqube ~]$ cd sonarqube-8.1.0.31237/
[sonar@sonarqube sonarqube-8.1.0.31237]$ cd bin/
[sonar@sonarqube bin]$ ls
jsw-license  linux-x86-64  macosx-universal-64  windows-x86-64
[sonar@sonarqube bin]$ cd linux-x86-64/
[sonar@sonarqube linux-x86-64]$ ls
lib  sonar.sh  wrapper
[sonar@sonarqube linux-x86-64]$ ./sonar.sh  status
SonarQube is not running.
[sonar@sonarqube linux-x86-64]$ ./sonar.sh  restart
Gracefully stopping SonarQube...
SonarQube was not running.
Starting SonarQube...
Started SonarQube.
[sonar@sonarqube linux-x86-64]$
[sonar@sonarqube linux-x86-64]$ ./sonar.sh  status
SonarQube is running (6018).
[sonar@sonarqube linux-x86-64]$
~~~


这时系统中将会运行多个 java 的进程

~~~
[sonar@sonarqube linux-x86-64]$ ps faux | grep -i java
sonar     6427  0.0  0.0  12520   996 pts/1    S+   15:56   0:00  |                                   \_ grep --color=auto -i java
sonar     6020  0.8  4.6 2646800 135644 ?      Sl   15:38   0:09  \_ java -Dsonar.wrapped=true -Djava.awt.headless=true -Xms8m -Xmx32m -Djava.library.path=./lib -classpath ../../lib/jsw/wrapper-3.2.3.jar:../../lib/common/woodstox-core-lgpl-4.4.0.jar:../../lib/common/woodstox-core-5.0.3.jar:../../lib/common/stax2-api-3.1.4.jar:../../lib/common/commons-csv-1.7.jar:../../lib/common/elasticsearch-secure-sm-6.8.4.jar:../../lib/common/lucene-core-7.7.2.jar:../../lib/common/lucene-analyzers-common-7.7.2.jar:../../lib/common/lucene-backward-codecs-7.7.2.jar:../../lib/common/lucene-grouping-7.7.2.jar:../../lib/common/lucene-highlighter-7.7.2.jar:../../lib/common/lucene-join-7.7.2.jar:../../lib/common/lucene-memory-7.7.2.jar:../../lib/common/lucene-misc-7.7.2.jar:../../lib/common/lucene-queries-7.7.2.jar:../../lib/common/lucene-queryparser-7.7.2.jar:../../lib/common/lucene-sandbox-7.7.2.jar:../../lib/common/lucene-spatial-7.7.2.jar:../../lib/common/lucene-spatial-extras-7.7.2.jar:../../lib/common/lucene-spatial3d-7.7.2.jar:../../lib/common/lucene-suggest-7.7.2.jar:../../lib/common/hppc-0.7.1.jar:../../lib/common/joda-time-2.10.3.jar:../../lib/common/t-digest-3.2.jar:../../lib/common/HdrHistogram-2.1.9.jar:../../lib/common/jna-4.5.1.jar:../../lib/common/netty-buffer-4.1.32.Final.jar:../../lib/common/netty-codec-4.1.32.Final.jar:../../lib/common/netty-codec-http-4.1.32.Final.jar:../../lib/common/netty-handler-4.1.32.Final.jar:../../lib/common/netty-resolver-4.1.32.Final.jar:../../lib/common/netty-transport-4.1.32.Final.jar:../../lib/common/failureaccess-1.0.1.jar:../../lib/common/listenablefuture-9999.0-empty-to-avoid-conflict-with-guava.jar:../../lib/common/jsr305-3.0.2.jar:../../lib/common/checker-qual-2.8.1.jar:../../lib/common/error_prone_annotations-2.3.2.jar:../../lib/common/j2objc-annotations-1.3.jar:../../lib/common/animal-sniffer-annotations-1.18.jar:../../lib/common/commons-pool2-2.7.0.jar:../../lib/common/commons-logging-1.2.jar:../../lib/common/core-3.1.0.jar:../../lib/common/diffutils-1.2.jar:../../lib/common/mybatis-3.5.3.jar:../../lib/common/commons-email-1.5.jar:../../lib/common/okhttp-3.14.2.jar:../../lib/common/tomcat-annotations-api-8.5.41.jar:../../lib/common/sonar-auth-common-8.1.0.31237.jar:../../lib/common/scribejava-apis-6.8.1.jar:../../lib/common/scribejava-core-6.8.1.jar:../../lib/common/commons-dbutils-1.5.jar:../../lib/common/jjwt-impl-0.10.7.jar:../../lib/common/jjwt-jackson-0.10.7.jar:../../lib/common/jjwt-api-0.10.7.jar:../../lib/common/jbcrypt-0.4.jar:../../lib/common/jackson-databind-2.9.9.2.jar:../../lib/common/jackson-core-2.9.9.jar:../../lib/common/jackson-dataformat-smile-2.8.11.jar:../../lib/common/jackson-dataformat-yaml-2.8.11.jar:../../lib/common/jackson-dataformat-cbor-2.8.11.jar:../../lib/common/jopt-simple-5.0.2.jar:../../lib/common/javax.mail-1.5.6.jar:../../lib/common/okio-1.17.2.jar:../../lib/common/lz4-1.3.0.jar:../../lib/common/commons-lang3-3.4.jar:../../lib/common/httpcore-4.4.12.jar:../../lib/common/activation-1.1.jar:../../lib/common/jackson-annotations-2.9.9.jar:../../lib/common/sonar-main-8.1.0.31237.jar:../../lib/common/sonar-ce-8.1.0.31237.jar:../../lib/common/sonar-webserver-8.1.0.31237.jar:../../lib/common/sonar-webserver-core-8.1.0.31237.jar:../../lib/common/sonar-ce-task-projectanalysis-8.1.0.31237.jar:../../lib/common/sonar-webserver-webapi-8.1.0.31237.jar:../../lib/common/sonar-ce-common-8.1.0.31237.jar:../../lib/common/sonar-ce-task-8.1.0.31237.jar:../../lib/common/sonar-webserver-ws-8.1.0.31237.jar:../../lib/common/sonar-webserver-es-8.1.0.31237.jar:../../lib/common/sonar-webserver-auth-8.1.0.31237.jar:../../lib/common/sonar-webserver-api-8.1.0.31237.jar:../../lib/common/sonar-server-common-8.1.0.31237.jar:../../lib/common/sonar-db-dao-8.1.0.31237.jar:../../lib/common/sonar-db-migration-8.1.0.31237.jar:../../lib/common/sonar-db-core-8.1.0.31237.jar:../../lib/common/sonar-process-8.1.0.31237.jar:../../lib/common/sonar-scanner-protocol-8.1.0.31237.jar:../../lib/common/sonar-core-8.1.0.31237.jar:../../lib/common/logback-classic-1.2.3.jar:../../lib/common/log4j-to-slf4j-2.8.2.jar:../../lib/common/jul-to-slf4j-1.7.28.jar:../../lib/common/sonar-update-center-common-1.23.0.723.jar:../../lib/common/sonar-auth-saml-8.1.0.31237.jar:../../lib/common/java-saml-2.5.0.jar:../../lib/common/java-saml-core-2.5.0.jar:../../lib/common/sonar-duplications-8.1.0.31237.jar:../../lib/common/sonar-markdown-8.1.0.31237.jar:../../lib/common/sonar-channel-4.1.jar:../../lib/common/xmlsec-2.1.4.jar:../../lib/common/slf4j-api-1.7.28.jar:../../lib/common/transport-6.8.4.jar:../../lib/common/sonar-plugin-api-impl-8.1.0.31237.jar:../../lib/common/sonar-plugin-api-8.1.0.31237-all.jar:../../lib/common/elasticsearch-6.8.4.jar:../../lib/common/transport-netty4-client-6.8.4.jar:../../lib/common/percolator-client-6.8.4.jar:../../lib/common/parent-join-client-6.8.4.jar:../../lib/common/guava-28.1-jre.jar:../../lib/common/sonar-ws-8.1.0.31237.jar:../../lib/common/protobuf-java-3.10.0.jar:../../lib/common/hazelcast-3.12.3.jar:../../lib/common/commons-io-2.6.jar:../../lib/common/commons-dbcp2-2.7.0.jar:../../lib/common/nanohttpd-2.3.0.jar:../../lib/common/picocontainer-2.15.jar:../../lib/common/logback-access-1.2.3.jar:../../lib/common/logback-core-1.2.3.jar:../../lib/common/sonar-auth-ldap-8.1.0.31237.jar:../../lib/common/commons-lang-2.6.jar:../../lib/common/netty-common-4.1.32.Final.jar:../../lib/common/log4j-api-2.8.2.jar:../../lib/common/elasticsearch-x-content-6.8.4.jar:../../lib/common/elasticsearch-cli-6.8.4.jar:../../lib/common/elasticsearch-core-6.8.4.jar:../../lib/common/snakeyaml-1.25.jar:../../lib/common/httpclient-4.5.10.jar:../../lib/common/commons-codec-1.12.jar:../../lib/common/sonar-auth-github-8.1.0.31237.jar:../../lib/common/sonar-auth-gitlab-8.1.0.31237.jar:../../lib/common/gson-2.8.5.jar:../../lib/common/tomcat-embed-core-8.5.41.jar:../../lib/common/sonar-classloader-1.0.jar:../../lib/common/staxmate-2.0.1.jar:../../lib/sonar-application-8.1.0.31237.jar:../../lib/sonar-shutdowner-8.1.0.31237.jar -Dwrapper.key=k_PtXmTkYwniQjy1 -Dwrapper.port=32000 -Dwrapper.jvm.port.min=31000 -Dwrapper.jvm.port.max=31999 -Dwrapper.pid=6018 -Dwrapper.version=3.2.3 -Dwrapper.native_library=wrapper -Dwrapper.service=TRUE -Dwrapper.cpu.timeout=10 -Dwrapper.jvmid=1 org.tanukisoftware.wrapper.WrapperSimpleApp org.sonar.application.App
sonar     6048  3.7 28.7 3247296 837332 ?      Sl   15:38   0:40      \_ /usr/lib/jvm/java-11-openjdk-11.0.5.10-0.el7_7.x86_64/bin/java -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=75 -XX:+UseCMSInitiatingOccupancyOnly -Des.networkaddress.cache.ttl=60 -Des.networkaddress.cache.negative.ttl=10 -XX:+AlwaysPreTouch -Xss1m -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djna.nosys=true -XX:-OmitStackTraceInFastThrow -Dio.netty.noUnsafe=true -Dio.netty.noKeySetOptimization=true -Dio.netty.recycler.maxCapacityPerThread=0 -Dlog4j.shutdownHookEnabled=false -Dlog4j2.disable.jmx=true -Djava.io.tmpdir=/home/sonar/sonarqube-8.1.0.31237/temp -XX:ErrorFile=../logs/es_hs_err_pid%p.log -Des.enforce.bootstrap.checks=true -Xmx512m -Xms512m -XX:+HeapDumpOnOutOfMemoryError -Des.path.home=/home/sonar/sonarqube-8.1.0.31237/elasticsearch -Des.path.conf=/home/sonar/sonarqube-8.1.0.31237/temp/conf/es -Des.distribution.flavor=default -Des.distribution.type=tar -cp /home/sonar/sonarqube-8.1.0.31237/elasticsearch/lib/* org.elasticsearch.bootstrap.Elasticsearch
sonar     6141  3.6 16.5 3281272 482268 ?      Sl   15:38   0:39      \_ /usr/lib/jvm/java-11-openjdk-11.0.5.10-0.el7_7.x86_64/bin/java -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djava.io.tmpdir=/home/sonar/sonarqube-8.1.0.31237/temp --add-opens=java.base/java.util=ALL-UNNAMED --add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED -Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Dhttp.nonProxyHosts=localhost|127.*|[::1] -cp ./lib/common/*:/home/sonar/sonarqube-8.1.0.31237/lib/jdbc/postgresql/postgresql-42.2.8.jar org.sonar.server.app.WebServer /home/sonar/sonarqube-8.1.0.31237/temp/sq-process14886157851691661885properties
sonar     6232  1.5 12.9 3231948 376640 ?      Sl   15:39   0:16      \_ /usr/lib/jvm/java-11-openjdk-11.0.5.10-0.el7_7.x86_64/bin/java -Djava.awt.headless=true -Dfile.encoding=UTF-8 -Djava.io.tmpdir=/home/sonar/sonarqube-8.1.0.31237/temp --add-opens=java.base/java.util=ALL-UNNAMED -Xmx512m -Xms128m -XX:+HeapDumpOnOutOfMemoryError -Dhttp.nonProxyHosts=localhost|127.*|[::1] -cp ./lib/common/*:/home/sonar/sonarqube-8.1.0.31237/lib/jdbc/postgresql/postgresql-42.2.8.jar org.sonar.ce.app.CeServer /home/sonar/sonarqube-8.1.0.31237/temp/sq-process13964334396696896338properties
[sonar@sonarqube linux-x86-64]$
~~~


## 日志文件

通过以下文件来跟踪启动过程中的错误和进度

~~~
[sonar@sonarqube ~]$ ll sonarqube-8.1.0.31237/logs/
total 176
-rw-r--r--. 1 sonar sonar     0 Jan 10 12:22 access.log
-rw-r--r--. 1 sonar sonar  1502 Jan 10 15:59 ce.log
-rw-r--r--. 1 sonar sonar 57642 Jan 10 15:59 es.log
-rw-r--r--. 1 sonar sonar    88 Dec 17 12:30 README.txt
-rw-r--r--. 1 sonar sonar 31612 Jan 10 15:59 sonar.log
-rw-r--r--. 1 sonar sonar 76083 Jan 10 15:59 web.log
[sonar@sonarqube ~]$
~~~

## 停止 sonarqube

~~~
[sonar@sonarqube linux-x86-64]$ ./sonar.sh stop
Gracefully stopping SonarQube...
Stopped SonarQube.
[sonar@sonarqube linux-x86-64]$ ps faux | grep -i java
sonar     6578  0.0  0.0  12520  1000 pts/1    S+   16:00   0:00  |                                   \_ grep --color=auto -i java
[sonar@sonarqube linux-x86-64]$
[sonar@sonarqube linux-x86-64]$ netstat  -ant
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:40251           0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:37183           0.0.0.0:*               LISTEN
tcp        0      0 10.0.2.15:22            10.0.2.2:54971          ESTABLISHED
tcp        0      0 10.0.2.15:22            10.0.2.2:54327          ESTABLISHED
tcp        0      0 10.0.2.15:22            10.0.2.2:54580          ESTABLISHED
tcp6       0      0 :::111                  :::*                    LISTEN
tcp6       0      0 :::40368                :::*                    LISTEN
tcp6       0      0 :::22                   :::*                    LISTEN
tcp6       0      0 ::1:25                  :::*                    LISTEN
tcp6       0      0 :::35551                :::*                    LISTEN
[sonar@sonarqube linux-x86-64]$
~~~


# 进行访问

~~~
[sonar@sonarqube linux-x86-64]$ ./sonar.sh status
SonarQube is not running.
[sonar@sonarqube linux-x86-64]$ ./sonar.sh start
Starting SonarQube...
Started SonarQube.
[sonar@sonarqube linux-x86-64]$
~~~

**`http://10.0.0.13:8000/sonarqube`**



![sonarqube](/assets/img/sonarqube/sonarqube01.png)


![sonarqube](/assets/img/sonarqube/sonarqube02.png)

默认密码为:

* login: admin
* password: admin

![sonarqube](/assets/img/sonarqube/sonarqube03.png)

![sonarqube](/assets/img/sonarqube/sonarqube04.png)

到此 sonarqube 的 server 端已经安装完成

如何使用，如果集成到 CI 中，留到后来来讲


---

# 总结

一个敏捷开发团队一定会需要一套静态代码扫描软件

**[SonarQube][sonarqube]** 就是一款非常受欢迎的代码扫描软件

上面的步骤描述了安装 **[SonarQube][sonarqube]** 的过程

后面接着讲如何扫描如何与 CI 集成

* TOC
{:toc}


---


[sonarqube]:https://www.sonarqube.org/
[sonarqube_install]:https://docs.sonarqube.org/latest/setup/install-server/
[requirements]:https://docs.sonarqube.org/latest/requirements/requirements/
[statictest]:https://www.ibm.com/developerworks/cn/java/j-lo-statictest-tools/