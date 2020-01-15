---
layout: post
title: "Integrate SonarQube to Jenkins"
date: 2020-01-14 02:43:48
image: '/assets/img/'
description: 'SonarQube 集成到 Jenkins'
main-class: tools
color: 
tags: 
 - maven
 - tools
 - jenkins
 - sonarqube
 - java
 - sdkman
 - gitlab
categories: 
 - sonarqube
twitter_text: 'Integrate SonarQube to Jenkins'
introduction: 'Integrate SonarQube to Jenkins'
---

# 前言

**[SonarQube][sonarqube]** 是一个很受欢迎的静态代码扫描软件

>Your teammate for Code Quality and Security
>SonarQube empowers all developers to write cleaner and safer code

能非常方便地集成到 CI/CD 环境中

这里分享一下 **[SonarQube][sonarqube]** 与 Jenkins 的集成方法

> **Tip:** 当前版本 **SonarQube Version: 8.1** Release on December 2019, **Jenkins ver. 2.214** 

---

# 操作

## 系统环境

~~~
[vagrant@jenkins ~]$ hostnamectl  
   Static hostname: jenkins
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 03699531e1dc431b849afd1e28341158
           Boot ID: 43317d66c0bf47288bd793b2c9cd71c8
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[vagrant@jenkins ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 81975sec preferred_lft 81975sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:db:e0:72 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.11/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fedb:e072/64 scope link 
       valid_lft forever preferred_lft forever
[vagrant@jenkins ~]$ tail -n 5 /etc/passwd
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:995::/var/lib/chrony:/sbin/nologin
vagrant:x:1000:1000:vagrant:/home/vagrant:/bin/bash
jenkins:x:997:994:Jenkins Automation Server:/var/lib/jenkins:/bin/false
[vagrant@jenkins ~]$
~~~


## 使用 Jenkins 的身份安装 Java 11

### 给 Jenkins 配置 bash

可以切换身份执行操作

~~~
[root@jenkins jenkins]# vim /etc/passwd
[root@jenkins jenkins]# sudo su - jenkins
-bash-4.2$ 
-bash-4.2$ pwd
/var/lib/jenkins
-bash-4.2$
~~~

~~~
[vagrant@jenkins ~]$ tail -n 2 /etc/passwd
vagrant:x:1000:1000:vagrant:/home/vagrant:/bin/bash
jenkins:x:997:994:Jenkins Automation Server:/var/lib/jenkins:/bin/bash
[vagrant@jenkins ~]$ 
~~~

### 安装 sdkman

~~~
[root@jenkins jenkins]# yum install -y install zip unzip 
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.01link.hk
 * extras: centos.01link.hk
 * updates: centos.01link.hk
No package install available.
Resolving Dependencies
--> Running transaction check
---> Package unzip.x86_64 0:6.0-20.el7 will be installed
---> Package zip.x86_64 0:3.0-11.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package          Arch              Version               Repository       Size
================================================================================
Installing:
 unzip            x86_64            6.0-20.el7            base            170 k
 zip              x86_64            3.0-11.el7            base            260 k

Transaction Summary
================================================================================
Install  2 Packages

Total download size: 430 k
Installed size: 1.1 M
Downloading packages:
(1/2): unzip-6.0-20.el7.x86_64.rpm                         | 170 kB   00:01     
(2/2): zip-3.0-11.el7.x86_64.rpm                           | 260 kB   00:01     
--------------------------------------------------------------------------------
Total                                              219 kB/s | 430 kB  00:01     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : zip-3.0-11.el7.x86_64                                        1/2 
  Installing : unzip-6.0-20.el7.x86_64                                      2/2 
  Verifying  : unzip-6.0-20.el7.x86_64                                      1/2 
  Verifying  : zip-3.0-11.el7.x86_64                                        2/2 

Installed:
  unzip.x86_64 0:6.0-20.el7               zip.x86_64 0:3.0-11.el7              

Complete!
[root@jenkins jenkins]#
[root@jenkins jenkins]# sudo su - jenkins
Last login: Tue Jan 14 03:15:08 UTC 2020 on pts/0
-bash-4.2$ curl -s "https://get.sdkman.io" | bash

                                -+syyyyyyys:
                            `/yho:`       -yd.
                         `/yh/`             +m.
                       .oho.                 hy                          .`
                     .sh/`                   :N`                `-/o`  `+dyyo:.
                   .yh:`                     `M-          `-/osysoym  :hs` `-+sys:      hhyssssssssy+
                 .sh:`                       `N:          ms/-``  yy.yh-      -hy.    `.N-````````+N.
               `od/`                         `N-       -/oM-      ddd+`     `sd:     hNNm        -N:
              :do`                           .M.       dMMM-     `ms.      /d+`     `NMMs       `do
            .yy-                             :N`    ```mMMM.      -      -hy.       /MMM:       yh
          `+d+`           `:/oo/`       `-/osyh/ossssssdNMM`           .sh:         yMMN`      /m.
         -dh-           :ymNMMMMy  `-/shmNm-`:N/-.``   `.sN            /N-         `NMMy      .m/
       `oNs`          -hysosmMMMMydmNmds+-.:ohm           :             sd`        :MMM/      yy
      .hN+           /d:    -MMMmhs/-.`   .MMMh   .ss+-                 `yy`       sMMN`     :N.
     :mN/           `N/     `o/-`         :MMMo   +MMMN-         .`      `ds       mMMh      do
    /NN/            `N+....--:/+oooosooo+:sMMM:   hMMMM:        `my       .m+     -MMM+     :N.
   /NMo              -+ooooo+/:-....`...:+hNMN.  `NMMMd`        .MM/       -m:    oMMN.     hs
  -NMd`                                    :mm   -MMMm- .s/     -MMm.       /m-   mMMd     -N.
 `mMM/                                      .-   /MMh. -dMo     -MMMy        od. .MMMs..---yh
 +MMM.                                           sNo`.sNMM+     :MMMM/        sh`+MMMNmNm+++-
 mMMM-                                           /--ohmMMM+     :MMMMm.       `hyymmmdddo
 MMMMh.                  ````                  `-+yy/`yMMM/     :MMMMMy       -sm:.``..-:-.`
 dMMMMmo-.``````..-:/osyhddddho.           `+shdh+.   hMMM:     :MmMMMM/   ./yy/` `:sys+/+sh/
 .dMMMMMMmdddddmmNMMMNNNNNMMMMMs           sNdo-      dMMM-  `-/yd/MMMMm-:sy+.   :hs-      /N`
  `/ymNNNNNNNmmdys+/::----/dMMm:          +m-         mMMM+ohmo/.` sMMMMdo-    .om:       `sh
     `.-----+/.`       `.-+hh/`         `od.          NMMNmds/     `mmy:`     +mMy      `:yy.
           /moyso+//+ossso:.           .yy`          `dy+:`         ..       :MMMN+---/oys:
         /+m:  `.-:::-`               /d+                                    +MMMMMMMNh:`
        +MN/                        -yh.                                     `+hddhy+.
       /MM+                       .sh:
      :NMo                      -sh/
     -NMs                    `/yy:
    .NMy                  `:sh+.
   `mMm`               ./yds-
  `dMMMmyo:-.````.-:oymNy:`
  +NMMMMMMMMMMMMMMMMms:`
    -+shmNMMMNmdy+:`


                                                                 Now attempting installation...


Looking for a previous installation of SDKMAN...
Looking for unzip...
Looking for zip...
Looking for curl...
Looking for sed...
Installing SDKMAN scripts...
Create distribution directories...
Getting available candidates...
Prime the config file...
Download script archive...
######################################################################## 100.0%
Extract script archive...
Install scripts...
Set version to 5.7.4+362 ...
Attempt update of interactive bash profile on regular UNIX...
Added sdkman init snippet to /var/lib/jenkins/.bashrc
Attempt update of zsh profile...
Updated existing /var/lib/jenkins/.zshrc



All done!


Please open a new terminal, or run the following in the existing one:

    source "/var/lib/jenkins/.sdkman/bin/sdkman-init.sh"

Then issue the following command:

    sdk help

Enjoy!!!
-bash-4.2$ source "/var/lib/jenkins/.sdkman/bin/sdkman-init.sh"
-bash-4.2$ sdk help
==== BROADCAST =================================================================
* 2020-01-13: Gradle 6.1-rc-3 released on SDKMAN! #gradle
* 2020-01-08: Asciidoctorj 2.2.0 released on SDKMAN! #asciidoctorj
* 2020-01-07: Gradle 6.1-rc-2 released on SDKMAN! #gradle
================================================================================

Usage: sdk <command> [candidate] [version]
       sdk offline <enable|disable>

   commands:
       install   or i    <candidate> [version] [local-path]
       uninstall or rm   <candidate> <version>
       list      or ls   [candidate]
       use       or u    <candidate> <version>
       default   or d    <candidate> [version]
       current   or c    [candidate]
       upgrade   or ug   [candidate]
       version   or v
       broadcast or b
       help      or h
       offline           [enable|disable]
       selfupdate        [force]
       update
       flush             <broadcast|archives|temp>

   candidate  :  the SDK to install: groovy, scala, grails, gradle, kotlin, etc.
                 use list command for comprehensive list of candidates
                 eg: $ sdk list
   version    :  where optional, defaults to latest stable if not provided
                 eg: $ sdk install groovy
   local-path :  optional path to an existing local installation
                 eg: $ sdk install groovy 2.4.13-local /opt/groovy-2.4.13

-bash-4.2$ 
~~~

给 Jenkins 用户安装 java 11 

~~~
-bash-4.2$ java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (build 1.8.0_232-b09)
OpenJDK 64-Bit Server VM (build 25.232-b09, mixed mode)
-bash-4.2$ 
-bash-4.2$ sdk ls java 
================================================================================
Available Java Versions
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 AdoptOpenJDK  |     | 13.0.1.j9    | adpt    |            | 13.0.1.j9-adpt      
               |     | 13.0.1.hs    | adpt    |            | 13.0.1.hs-adpt      
               |     | 12.0.2.j9    | adpt    |            | 12.0.2.j9-adpt      
               |     | 12.0.2.hs    | adpt    |            | 12.0.2.hs-adpt      
               |     | 11.0.5.j9    | adpt    |            | 11.0.5.j9-adpt      
               |     | 11.0.5.hs    | adpt    |            | 11.0.5.hs-adpt      
               |     | 8.0.232.j9   | adpt    |            | 8.0.232.j9-adpt     
               |     | 8.0.232.hs   | adpt    |            | 8.0.232.hs-adpt     
 Amazon        |     | 11.0.5       | amzn    |            | 11.0.5-amzn         
               |     | 8.0.232      | amzn    |            | 8.0.232-amzn        
 Azul Zulu     |     | 13.0.1       | zulu    |            | 13.0.1-zulu         
               |     | 12.0.2       | zulu    |            | 12.0.2-zulu         
               |     | 11.0.5       | zulu    |            | 11.0.5-zulu         
               |     | 10.0.2       | zulu    |            | 10.0.2-zulu         
               |     | 9.0.7        | zulu    |            | 9.0.7-zulu          
               |     | 8.0.232      | zulu    |            | 8.0.232-zulu        
               |     | 7.0.242      | zulu    |            | 7.0.242-zulu        
               |     | 6.0.119      | zulu    |            | 6.0.119-zulu        
 Azul ZuluFX   |     | 11.0.2       | zulufx  |            | 11.0.2-zulufx       
               |     | 8.0.202      | zulufx  |            | 8.0.202-zulufx      
 BellSoft      |     | 13.0.1       | librca  |            | 13.0.1-librca       
               |     | 12.0.2       | librca  |            | 12.0.2-librca       
               |     | 11.0.5       | librca  |            | 11.0.5-librca       
               |     | 8.0.232      | librca  |            | 8.0.232-librca      
 GraalVM       |     | 19.3.0.r11   | grl     |            | 19.3.0.r11-grl      
               |     | 19.3.0.r8    | grl     |            | 19.3.0.r8-grl       
               |     | 19.3.0.2.r11 | grl     |            | 19.3.0.2.r11-grl    
               |     | 19.3.0.2.r8  | grl     |            | 19.3.0.2.r8-grl     
               |     | 19.2.1       | grl     |            | 19.2.1-grl          
               |     | 19.1.1       | grl     |            | 19.1.1-grl          
               |     | 19.0.2       | grl     |            | 19.0.2-grl          
               |     | 1.0.0        | grl     |            | 1.0.0-rc-16-grl     
 Java.net      |     | 15.ea.4      | open    |            | 15.ea.4-open        
               |     | 14.ea.30     | open    |            | 14.ea.30-open       
               |     | 13.0.1       | open    |            | 13.0.1-open         
               |     | 12.0.2       | open    |            | 12.0.2-open         
               |     | 11.0.5       | open    |            | 11.0.5-open         
               |     | 10.0.2       | open    |            | 10.0.2-open         
               |     | 9.0.4        | open    |            | 9.0.4-open          
               |     | 8.0.232      | open    |            | 8.0.232-open        
 SAP           |     | 12.0.2       | sapmchn |            | 12.0.2-sapmchn      
               |     | 11.0.4       | sapmchn |            | 11.0.4-sapmchn      
================================================================================
Use the Identifier for installation:

    $ sdk install java 11.0.3.hs-adpt
================================================================================
-bash-4.2$ 
-bash-4.2$ sdk i java 11.0.5-open

Downloading: java 11.0.5-open

In progress...

######################################################################## 100.0%

Repackaging Java 11.0.5-open...

Done repackaging...

Installing: java 11.0.5-open
Done installing!


Setting java 11.0.5-open as default.
-bash-4.2$ java -version 
openjdk version "11.0.5" 2019-10-15
OpenJDK Runtime Environment 18.9 (build 11.0.5+10)
OpenJDK 64-Bit Server VM 18.9 (build 11.0.5+10, mixed mode)
-bash-4.2$ sdk ls java 
================================================================================
Available Java Versions
================================================================================
 Vendor        | Use | Version      | Dist    | Status     | Identifier
--------------------------------------------------------------------------------
 AdoptOpenJDK  |     | 13.0.1.j9    | adpt    |            | 13.0.1.j9-adpt      
               |     | 13.0.1.hs    | adpt    |            | 13.0.1.hs-adpt      
               |     | 12.0.2.j9    | adpt    |            | 12.0.2.j9-adpt      
               |     | 12.0.2.hs    | adpt    |            | 12.0.2.hs-adpt      
               |     | 11.0.5.j9    | adpt    |            | 11.0.5.j9-adpt      
               |     | 11.0.5.hs    | adpt    |            | 11.0.5.hs-adpt      
               |     | 8.0.232.j9   | adpt    |            | 8.0.232.j9-adpt     
               |     | 8.0.232.hs   | adpt    |            | 8.0.232.hs-adpt     
 Amazon        |     | 11.0.5       | amzn    |            | 11.0.5-amzn         
               |     | 8.0.232      | amzn    |            | 8.0.232-amzn        
 Azul Zulu     |     | 13.0.1       | zulu    |            | 13.0.1-zulu         
               |     | 12.0.2       | zulu    |            | 12.0.2-zulu         
               |     | 11.0.5       | zulu    |            | 11.0.5-zulu         
               |     | 10.0.2       | zulu    |            | 10.0.2-zulu         
               |     | 9.0.7        | zulu    |            | 9.0.7-zulu          
               |     | 8.0.232      | zulu    |            | 8.0.232-zulu        
               |     | 7.0.242      | zulu    |            | 7.0.242-zulu        
               |     | 6.0.119      | zulu    |            | 6.0.119-zulu        
 Azul ZuluFX   |     | 11.0.2       | zulufx  |            | 11.0.2-zulufx       
               |     | 8.0.202      | zulufx  |            | 8.0.202-zulufx      
 BellSoft      |     | 13.0.1       | librca  |            | 13.0.1-librca       
               |     | 12.0.2       | librca  |            | 12.0.2-librca       
               |     | 11.0.5       | librca  |            | 11.0.5-librca       
               |     | 8.0.232      | librca  |            | 8.0.232-librca      
 GraalVM       |     | 19.3.0.r11   | grl     |            | 19.3.0.r11-grl      
               |     | 19.3.0.r8    | grl     |            | 19.3.0.r8-grl       
               |     | 19.3.0.2.r11 | grl     |            | 19.3.0.2.r11-grl    
               |     | 19.3.0.2.r8  | grl     |            | 19.3.0.2.r8-grl     
               |     | 19.2.1       | grl     |            | 19.2.1-grl          
               |     | 19.1.1       | grl     |            | 19.1.1-grl          
               |     | 19.0.2       | grl     |            | 19.0.2-grl          
               |     | 1.0.0        | grl     |            | 1.0.0-rc-16-grl     
 Java.net      |     | 15.ea.4      | open    |            | 15.ea.4-open        
               |     | 14.ea.30     | open    |            | 14.ea.30-open       
               |     | 13.0.1       | open    |            | 13.0.1-open         
               |     | 12.0.2       | open    |            | 12.0.2-open         
               | >>> | 11.0.5       | open    | installed  | 11.0.5-open         
               |     | 10.0.2       | open    |            | 10.0.2-open         
               |     | 9.0.4        | open    |            | 9.0.4-open          
               |     | 8.0.232      | open    |            | 8.0.232-open        
 SAP           |     | 12.0.2       | sapmchn |            | 12.0.2-sapmchn      
               |     | 11.0.4       | sapmchn |            | 11.0.4-sapmchn      
================================================================================
Use the Identifier for installation:

    $ sdk install java 11.0.3.hs-adpt
================================================================================
-bash-4.2$ 
-bash-4.2$ cd 
-bash-4.2$ pwd
/var/lib/jenkins
-bash-4.2$ which java
~/.sdkman/candidates/java/current/bin/java
-bash-4.2$ ll ~/.sdkman/candidates/java
total 0
drwxrwxr-x. 10 jenkins jenkins 119 Oct  9 11:00 11.0.5-open
lrwxrwxrwx.  1 jenkins jenkins  52 Jan 14 03:46 current -> /var/lib/jenkins/.sdkman/candidates/java/11.0.5-open
-bash-4.2$
~~~

## Jenkins 中安装 sonarqube 插件

### 安装插件

安装 sonarqube 插件

![sonarqube](/assets/img/sonarqube/sonarqube10.png)

![sonarqube](/assets/img/sonarqube/sonarqube11.png)

![sonarqube](/assets/img/sonarqube/sonarqube12.png)

使用一样的方式安装上 **`Maven Integration plugin, GitLab Plugin, 	
Gitlab Hook Plugin`** 等插件

### 配置 sonarqube Server 信息

![sonarqube](/assets/img/sonarqube/sonarqube13.png)

![sonarqube](/assets/img/sonarqube/sonarqube14.png)

### 配置 Java

![sonarqube](/assets/img/sonarqube/sonarqube15.png) 

### 配置 Maven

![sonarqube](/assets/img/sonarqube/sonarqube16.png)

这样当系统中要调用 Maven 的时候，就会自动下载

### 配置 SonarQube Scanner

![sonarqube](/assets/img/sonarqube/sonarqube17.png)


## 添加 RSA key 到 gitlab 中

~~~

[vagrant@workspace ~]$ ssh-keygen -t rsa 
Generating public/private rsa key pair.
Enter file in which to save the key (/home/vagrant/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/vagrant/.ssh/id_rsa.
Your public key has been saved in /home/vagrant/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:z9udhPsvmoKoRJxe3aaKjx10VDwJEfInQnXnT2L6ESs vagrant@workspace
The key's randomart image is:
+---[RSA 2048]----+
|      o.==o..    |
|     . o o+o     |
|      . + ..= .  |
|   . . + + o *   |
|    + o S E o .  |
|   o o . = o o   |
|    o ....o o .  |
|   . +.o. .o =.. |
|    +o+   ..=o+o.|
+----[SHA256]-----+
[vagrant@workspace ~]$ 
[vagrant@workspace ~]$ cd .ssh/
[vagrant@workspace .ssh]$ ls
authorized_keys  id_rsa  id_rsa.pub
[vagrant@workspace .ssh]$ cat id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDD25M2p93F2TdDNH6mpu+0tMsz7sLOkfivcGP2TgCw++xZRB8L7rfebbqZ22I6+IrcW0gkQc2CLk8cJsKAH07TL7ea5w2NE6N9/0IXGs8WFilevUUysXGMyaFlcaDIVSPwTjTOaL0rfQPIWDNmK840B7byR50Um38/s+s44GcO3tS5mHW2ISq5SuyHKJaO6TKOH2j1YaKD87i/uVXyPRWQB4KM5b7tZuRPzVSbGWKTDRu4zbUkHoZuUNUF5ttnfASgXRurTfvt6WBzwPIBVEPtdh2dh+aFybYcoRqxgQVrXCNClAiI0HGbJa9wS5xCZYQGRUaxiCht9RjKiRt3L9K3 vagrant@workspace
[vagrant@workspace .ssh]$ 
~~~

在 Gitlab 里添加上这个 public key

![sonarqube](/assets/img/sonarqube/sonarqube18.png)

## 创建新的库

![sonarqube](/assets/img/sonarqube/sonarqube19.png)

并且推送代码

~~~
[vagrant@workspace ~]$ unzip maven-plugin.zip 
Archive:  maven-plugin.zip
   creating: maven-plugin/
   creating: maven-plugin/target/
  inflating: maven-plugin/pom.xml    
  inflating: maven-plugin/.classpath  
  inflating: maven-plugin/.project   
   creating: maven-plugin/src/
   creating: maven-plugin/src/test/
   creating: maven-plugin/src/test/java/
   creating: maven-plugin/src/main/
   creating: maven-plugin/src/main/java/
   creating: maven-plugin/src/main/java/com/
   creating: maven-plugin/src/main/java/com/itranswarp/
   creating: maven-plugin/src/main/java/com/itranswarp/learnjava/
  inflating: maven-plugin/src/main/java/com/itranswarp/learnjava/Main.java  
[vagrant@workspace ~]$ ls
maven-plugin  maven-plugin.zip  x.zip
[vagrant@workspace ~]$ cd maven-plugin/
[vagrant@workspace maven-plugin]$ git config --global user.name "yyghdfz"
[vagrant@workspace maven-plugin]$ git config --global user.email "yyghdfz@163.com"
[vagrant@workspace maven-plugin]$ git init
Initialized empty Git repository in /home/vagrant/maven-plugin/.git/
[vagrant@workspace maven-plugin]$ git remote add origin git@10.0.0.12:yyghdfz/hello.git
[vagrant@workspace maven-plugin]$ git add .
[vagrant@workspace maven-plugin]$ git commit -m "Initial commit"
[master (root-commit) fb5952c] Initial commit
 4 files changed, 128 insertions(+)
 create mode 100644 .classpath
 create mode 100644 .project
 create mode 100644 pom.xml
 create mode 100644 src/main/java/com/itranswarp/learnjava/Main.java
[vagrant@workspace maven-plugin]$ git push -u origin master
Counting objects: 12, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (6/6), done.
Writing objects: 100% (12/12), 1.94 KiB | 0 bytes/s, done.
Total 12 (delta 0), reused 0 (delta 0)
To git@10.0.0.12:yyghdfz/hello.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
[vagrant@workspace maven-plugin]$ 
[vagrant@workspace maven-plugin]$ ls
pom.xml  src  target
[vagrant@workspace maven-plugin]$ tree
.
|-- pom.xml
|-- src
|   |-- main
|   |   `-- java
|   |       `-- com
|   |           `-- itranswarp
|   |               `-- learnjava
|   |                   `-- Main.java
|   `-- test
|       `-- java
`-- target

9 directories, 2 files
[vagrant@workspace maven-plugin]$ 
~~~


>**Tip:** 要在 Jenkins 中安装上 Gitlab 的插件和 Gitlab Hook 插件



## 创建一对帐密

![sonarqube](/assets/img/sonarqube/sonarqube20.png)


## 配置一个自由项目

SCM 配置到 gitlab 中的 demo 库地址

使用前面配置的账密对

![sonarqube](/assets/img/sonarqube/sonarqube21.png)

使用 Trigger 的方式来触发构建

(这种方式比定期轮询更为高效)

![sonarqube](/assets/img/sonarqube/sonarqube22.png)

加了加强安全，可以生成 Secret key，过滤掉未经授权的触发

![sonarqube](/assets/img/sonarqube/sonarqube23.png)

加上 SonarQube scanner 的配置

这里要留意项目名称 **sonar.projectKey** 是必填字段，否则会报找不到此信息的错误


类似如下

~~~
...
...
...
ERROR: Error during SonarQube Scanner execution
ERROR: You must define the following mandatory properties for 'Unknown': sonar.projectKey
ERROR: 
ERROR: Re-run SonarQube Scanner using the -X switch to enable full debug logging.
WARN: Unable to locate 'report-task.txt' in the workspace. Did the SonarScanner succeeded?
ERROR: SonarQube scanner exited with non-zero code: 2
Finished: FAILURE
~~~

同时这个文件可以放到代码库中一同管理起来, 文件名设定为 **sonar-project.properties**, scanner 会自动去读，这样配置就有了版本管理

可以参考 **[SonarScanner][sonarscanner]**

![sonarqube](/assets/img/sonarqube/sonarqube24.png)

调用 Maven 插件进行构建

![sonarqube](/assets/img/sonarqube/sonarqube25.png)


## 配置 gitlab webhook

使用管理员的身份打开 **Allow requests to the local network from hooks and services**

![sonarqube](/assets/img/sonarqube/sonarqube26.png)

否则当目标路径在本地网络的时候，会有如下报错

![sonarqube](/assets/img/sonarqube/sonarqube27.png)

打开后，再添加就会成功

![sonarqube](/assets/img/sonarqube/sonarqube28.png)

可以直接点击测试来 trigger jenkins build 

![sonarqube](/assets/img/sonarqube/sonarqube29.png)

可以看到 Jenkins 这边已经触发了

![sonarqube](/assets/img/sonarqube/sonarqube30.png)


## 通过 push 代码触发构建

在本地作一个小的改动，push 一下代码

~~~
(base) [vagrant@workspace maven-plugin]$ vim src/main/java/com/itranswarp/learnjava/Main.java
(base) [vagrant@workspace maven-plugin]$ git diff
diff --git a/src/main/java/com/itranswarp/learnjava/Main.java b/src/main/java/com/itranswarp/learnjava/Main.java
index 991f5c4..0261007 100644
--- a/src/main/java/com/itranswarp/learnjava/Main.java
+++ b/src/main/java/com/itranswarp/learnjava/Main.java
@@ -12,7 +12,7 @@ public class Main {
        public static void main(String[] args) throws Exception {
                var logger = LoggerFactory.getLogger(Main.class);
                logger.info("start application...");
-               for (int i = 1; i <= 99; i++) {
+               for (int i = 1; i <= 22; i++) {
                        Thread.sleep(100);
                        logger.warn("begin task {}...", i);
                }
(base) [vagrant@workspace maven-plugin]$ git add . ;
(base) [vagrant@workspace maven-plugin]$ git commit -m "99 to 22"
[master 2232c29] 99 to 22
 1 file changed, 1 insertion(+), 1 deletion(-)
(base) [vagrant@workspace maven-plugin]$ git push
warning: push.default is unset; its implicit value is changing in
Git 2.0 from 'matching' to 'simple'. To squelch this message
and maintain the current behavior after the default changes, use:

  git config --global push.default matching

To squelch this message and adopt the new behavior now, use:

  git config --global push.default simple

See 'git help config' and search for 'push.default' for further information.
(the 'simple' mode was introduced in Git 1.7.11. Use the similar mode
'current' instead of 'simple' if you sometimes use older versions of Git)

Counting objects: 17, done.
Delta compression using up to 2 threads.
Compressing objects: 100% (3/3), done.
Writing objects: 100% (9/9), 531 bytes | 0 bytes/s, done.
Total 9 (delta 2), reused 0 (delta 0)
To git@10.0.0.12:yyghdfz/hello.git
   a7d8dda..2232c29  master -> master
(base) [vagrant@workspace maven-plugin]$
~~~

这时 Jenkins 里的这个任务就直接被触发了

![sonarqube](/assets/img/sonarqube/sonarqube31.png)

构建完成后，可以直接点击 SonarQube 的图标对扫描结果进行查看

![sonarqube](/assets/img/sonarqube/sonarqube32.png)

进入项目，可以看到日志输出

![sonarqube](/assets/img/sonarqube/sonarqube33.png)


到此 SonarQube 与 Jenkins 的集成就演示完毕了

>**Tip:** 生产中，为了便于开发人员在本地进行测试，有一种实践是将 SonarScanner 直接集成到本地的 Maven 中，这样在本地可以直接生成报告，CI环境中也由 Jenkins 调用 Maven 来进行构建，这样本地和 CI 中都可以用同一种方式生成 SonarQube 的报告

---

# 总结

一个敏捷开发团队一定会需要一套静态代码扫描软件

**[SonarQube][sonarqube]** 就是这样的软件

上面的步骤描述了 **[SonarQube][sonarqube]** 与 Jenkins 的集成过程

后面可以将上述过程在 Jenkins 里改造成 Pipeline 

* TOC
{:toc}


---

[sonarqube]:https://www.sonarqube.org/
[sonarscanner_for_maven]:https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-maven/
[sonarscanner]:https://docs.sonarqube.org/latest/analysis/scan/sonarscanner/
