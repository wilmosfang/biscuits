---
layout: post
title: "WordPress"
date: 2019-08-24 01:23:42
image: '/assets/img/'
description: '安装 WordPress'
main-class: 'php'
color: '#767bb3'
tags:
 - linux
 - apache
 - mysql
 - php
 - lamp
 - wordpress
categories:
 - php
twitter_text: 'simple process of WordPress installation'
introduction: 'Installation of WordPress'
---



# 前言

**[WordPress][wordpress]** 是一款用 php 实现的开源 CMS 软件

>WordPress is open source software you can use to create a beautiful website, blog, or app

因为特性丰富，使用简单，它在个人博客领域非常受欢迎

在技术型组织里，可以使用它来构建一个文档管理系统或者信息发布平台

这里演示一下如何构建 **[WordPress][wordpress]**

参考 **[Installing WordPress][wordpress_install]**

> **Tip:** 当前的版本为 **WordPress 5.2.2**

---

# 操作

## 依赖

* PHP version 7.3 or greater.
* MySQL version 5.6 or greater OR MariaDB version 10.1 or greater.
* HTTPS support

当前版本的 WordPress 需要满足以上的运行环境

>**Tip:** 详细可以参考 **[Requirements][wordpress_dep]**

## OS 环境

~~~
[root@wordpress ~]# hostnamectl 
   Static hostname: wordpress
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 275f9338e94e451889a5e3f73980a170
           Boot ID: f1dd7605eb0c43de954455c27e151910
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@wordpress ~]# cat /etc/centos-release
CentOS Linux release 7.6.1810 (Core) 
[root@wordpress ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86353sec preferred_lft 86353sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:c0:c1:52 brd ff:ff:ff:ff:ff:ff
    inet 10.1.0.200/24 brd 10.1.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fec0:c152/64 scope link 
       valid_lft forever preferred_lft forever
[root@wordpress ~]# 
~~~


## 安装数据库

### 配置 mariadb 仓库

~~~
[root@wordpress ~]# vim /etc/yum.repos.d/mariadb.repo
[root@wordpress ~]# cat /etc/yum.repos.d/mariadb.repo
# MariaDB 10.4 CentOS repository list - created 2019-08-24 01:55 UTC
# http://downloads.mariadb.org/mariadb/repositories/
[mariadb]
name = MariaDB
baseurl = http://yum.mariadb.org/10.4/centos7-amd64
gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck=1
[root@wordpress ~]# 
~~~

### 安装数据库

~~~
[root@wordpress ~]# sudo yum install MariaDB-server MariaDB-client
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.cqu.edu.cn
 * updates: mirrors.huaweicloud.com
mariadb                                                  | 2.9 kB     00:00     
mariadb/primary_db                                         |  42 kB   00:01     
Resolving Dependencies
--> Running transaction check
---> Package MariaDB-client.x86_64 0:10.4.7-1.el7.centos will be installed
--> Processing Dependency: MariaDB-common for package: MariaDB-client-10.4.7-1.el7.centos.x86_64
--> Processing Dependency: libaio.so.1(LIBAIO_0.4)(64bit) for package: MariaDB-client-10.4.7-1.el7.centos.x86_64
--> Processing Dependency: libaio.so.1(LIBAIO_0.1)(64bit) for package: MariaDB-client-10.4.7-1.el7.centos.x86_64
--> Processing Dependency: libaio.so.1()(64bit) for package: MariaDB-client-10.4.7-1.el7.centos.x86_64
---> Package MariaDB-server.x86_64 0:10.4.7-1.el7.centos will be installed
--> Processing Dependency: perl(DBI) for package: MariaDB-server-10.4.7-1.el7.centos.x86_64
--> Processing Dependency: galera-4 for package: MariaDB-server-10.4.7-1.el7.centos.x86_64
--> Processing Dependency: lsof for package: MariaDB-server-10.4.7-1.el7.centos.x86_64
--> Processing Dependency: perl(Data::Dumper) for package: MariaDB-server-10.4.7-1.el7.centos.x86_64
--> Running transaction check
---> Package MariaDB-common.x86_64 0:10.4.7-1.el7.centos will be installed
--> Processing Dependency: MariaDB-compat for package: MariaDB-common-10.4.7-1.el7.centos.x86_64
---> Package galera-4.x86_64 0:26.4.2-1.rhel7.el7.centos will be installed
--> Processing Dependency: libboost_program_options.so.1.53.0()(64bit) for package: galera-4-26.4.2-1.rhel7.el7.centos.x86_64
---> Package libaio.x86_64 0:0.3.109-13.el7 will be installed
---> Package lsof.x86_64 0:4.87-6.el7 will be installed
---> Package perl-DBI.x86_64 0:1.627-4.el7 will be installed
--> Processing Dependency: perl(RPC::PlServer) >= 0.2001 for package: perl-DBI-1.627-4.el7.x86_64
--> Processing Dependency: perl(RPC::PlClient) >= 0.2000 for package: perl-DBI-1.627-4.el7.x86_64
---> Package perl-Data-Dumper.x86_64 0:2.145-3.el7 will be installed
--> Running transaction check
---> Package MariaDB-compat.x86_64 0:10.4.7-1.el7.centos will be obsoleting
---> Package boost-program-options.x86_64 0:1.53.0-27.el7 will be installed
---> Package mariadb-libs.x86_64 1:5.5.60-1.el7_5 will be obsoleted
---> Package perl-PlRPC.noarch 0:0.2020-14.el7 will be installed
--> Processing Dependency: perl(Net::Daemon) >= 0.13 for package: perl-PlRPC-0.2020-14.el7.noarch
--> Processing Dependency: perl(Net::Daemon::Test) for package: perl-PlRPC-0.2020-14.el7.noarch
--> Processing Dependency: perl(Net::Daemon::Log) for package: perl-PlRPC-0.2020-14.el7.noarch
--> Processing Dependency: perl(Compress::Zlib) for package: perl-PlRPC-0.2020-14.el7.noarch
--> Running transaction check
---> Package perl-IO-Compress.noarch 0:2.061-2.el7 will be installed
--> Processing Dependency: perl(Compress::Raw::Zlib) >= 2.061 for package: perl-IO-Compress-2.061-2.el7.noarch
--> Processing Dependency: perl(Compress::Raw::Bzip2) >= 2.061 for package: perl-IO-Compress-2.061-2.el7.noarch
---> Package perl-Net-Daemon.noarch 0:0.48-5.el7 will be installed
--> Running transaction check
---> Package perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7 will be installed
---> Package perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                   Arch     Version                     Repository
                                                                           Size
================================================================================
Installing:
 MariaDB-client            x86_64   10.4.7-1.el7.centos         mariadb    12 M
 MariaDB-compat            x86_64   10.4.7-1.el7.centos         mariadb   2.8 M
     replacing  mariadb-libs.x86_64 1:5.5.60-1.el7_5
 MariaDB-server            x86_64   10.4.7-1.el7.centos         mariadb    26 M
Installing for dependencies:
 MariaDB-common            x86_64   10.4.7-1.el7.centos         mariadb    81 k
 boost-program-options     x86_64   1.53.0-27.el7               base      156 k
 galera-4                  x86_64   26.4.2-1.rhel7.el7.centos   mariadb   9.4 M
 libaio                    x86_64   0.3.109-13.el7              base       24 k
 lsof                      x86_64   4.87-6.el7                  base      331 k
 perl-Compress-Raw-Bzip2   x86_64   2.061-3.el7                 base       32 k
 perl-Compress-Raw-Zlib    x86_64   1:2.061-4.el7               base       57 k
 perl-DBI                  x86_64   1.627-4.el7                 base      802 k
 perl-Data-Dumper          x86_64   2.145-3.el7                 base       47 k
 perl-IO-Compress          noarch   2.061-2.el7                 base      260 k
 perl-Net-Daemon           noarch   0.48-5.el7                  base       51 k
 perl-PlRPC                noarch   0.2020-14.el7               base       36 k

Transaction Summary
================================================================================
Install  3 Packages (+12 Dependent packages)

Total download size: 51 M
Is this ok [y/d/N]: y
Downloading packages:
warning: /var/cache/yum/x86_64/7/mariadb/packages/MariaDB-common-10.4.7-1.el7.centos.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 1bb943db: NOKEY
Public key for MariaDB-common-10.4.7-1.el7.centos.x86_64.rpm is not installed
(1/15): MariaDB-common-10.4.7-1.el7.centos.x86_64.rpm      |  81 kB   00:02     
(2/15): MariaDB-compat-10.4.7-1.el7.centos.x86_64.rpm      | 2.8 MB   00:58     
(3/15): boost-program-options-1.53.0-27.el7.x86_64.rpm     | 156 kB   00:11     
(4/15): MariaDB-client-10.4.7-1.el7.centos.x86_64.rpm      |  12 MB   05:30     
(5/15): perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64.rpm     |  32 kB   00:00     
(6/15): libaio-0.3.109-13.el7.x86_64.rpm                   |  24 kB   00:00     
(7/15): perl-Compress-Raw-Zlib-2.061-4.el7.x86_64.rpm      |  57 kB   00:00     
(8/15): perl-Data-Dumper-2.145-3.el7.x86_64.rpm            |  47 kB   00:00     
(9/15): perl-DBI-1.627-4.el7.x86_64.rpm                    | 802 kB   00:00     
(10/15): perl-Net-Daemon-0.48-5.el7.noarch.rpm             |  51 kB   00:00     
(11/15): perl-PlRPC-0.2020-14.el7.noarch.rpm               |  36 kB   00:00     
(12/15): perl-IO-Compress-2.061-2.el7.noarch.rpm           | 260 kB   00:00     
(13/15): lsof-4.87-6.el7.x86_64.rpm                        | 331 kB   00:02     
(14/15): galera-4-26.4.2-1.rhel7.el7.centos.x86_64.rpm     | 9.4 MB   03:25     
(15/15): MariaDB-server-10.4.7-1.el7.centos.x86_64.rpm     |  26 MB   13:40     
--------------------------------------------------------------------------------
Total                                               60 kB/s |  51 MB  14:42     
Retrieving key from https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
Importing GPG key 0x1BB943DB:
 Userid     : "MariaDB Package Signing Key <package-signing-key@mariadb.org>"
 Fingerprint: 1993 69e5 404b d5fc 7d2f e43b cbcb 082a 1bb9 43db
 From       : https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
Is this ok [y/N]: y
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libaio-0.3.109-13.el7.x86_64                                1/16 
  Installing : perl-Data-Dumper-2.145-3.el7.x86_64                         2/16 
  Installing : MariaDB-common-10.4.7-1.el7.centos.x86_64                   3/16 
  Installing : MariaDB-compat-10.4.7-1.el7.centos.x86_64                   4/16 
  Installing : MariaDB-client-10.4.7-1.el7.centos.x86_64                   5/16 
  Installing : boost-program-options-1.53.0-27.el7.x86_64                  6/16 
  Installing : galera-4-26.4.2-1.rhel7.el7.centos.x86_64                   7/16 
  Installing : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                  8/16 
  Installing : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                 9/16 
  Installing : perl-IO-Compress-2.061-2.el7.noarch                        10/16 
  Installing : lsof-4.87-6.el7.x86_64                                     11/16 
  Installing : perl-Net-Daemon-0.48-5.el7.noarch                          12/16 
  Installing : perl-PlRPC-0.2020-14.el7.noarch                            13/16 
  Installing : perl-DBI-1.627-4.el7.x86_64                                14/16 
  Installing : MariaDB-server-10.4.7-1.el7.centos.x86_64                  15/16 


Two all-privilege accounts were created.
One is root@localhost, it has no password, but you need to
be system 'root' user to connect. Use, for example, sudo mysql
The second is mysql@localhost, it has no password either, but
you need to be the system 'mysql' user to connect.
After connecting you can set the password, if you would need to be
able to connect as any of these users with a password and without sudo

See the MariaDB Knowledgebase at http://mariadb.com/kb or the
MySQL manual for more instructions.

Please report any problems at http://mariadb.org/jira

The latest information about MariaDB is available at http://mariadb.org/.
You can find additional information about the MySQL part at:
http://dev.mysql.com
Consider joining MariaDB's strong and vibrant community:
https://mariadb.org/get-involved/

  Erasing    : 1:mariadb-libs-5.5.60-1.el7_5.x86_64                       16/16 
  Verifying  : MariaDB-server-10.4.7-1.el7.centos.x86_64                   1/16 
  Verifying  : galera-4-26.4.2-1.rhel7.el7.centos.x86_64                   2/16 
  Verifying  : MariaDB-compat-10.4.7-1.el7.centos.x86_64                   3/16 
  Verifying  : perl-Net-Daemon-0.48-5.el7.noarch                           4/16 
  Verifying  : perl-Data-Dumper-2.145-3.el7.x86_64                         5/16 
  Verifying  : MariaDB-client-10.4.7-1.el7.centos.x86_64                   6/16 
  Verifying  : lsof-4.87-6.el7.x86_64                                      7/16 
  Verifying  : MariaDB-common-10.4.7-1.el7.centos.x86_64                   8/16 
  Verifying  : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                 9/16 
  Verifying  : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                 10/16 
  Verifying  : boost-program-options-1.53.0-27.el7.x86_64                 11/16 
  Verifying  : perl-DBI-1.627-4.el7.x86_64                                12/16 
  Verifying  : libaio-0.3.109-13.el7.x86_64                               13/16 
  Verifying  : perl-PlRPC-0.2020-14.el7.noarch                            14/16 
  Verifying  : perl-IO-Compress-2.061-2.el7.noarch                        15/16 
  Verifying  : 1:mariadb-libs-5.5.60-1.el7_5.x86_64                       16/16 

Installed:
  MariaDB-client.x86_64 0:10.4.7-1.el7.centos                                   
  MariaDB-compat.x86_64 0:10.4.7-1.el7.centos                                   
  MariaDB-server.x86_64 0:10.4.7-1.el7.centos                                   

Dependency Installed:
  MariaDB-common.x86_64 0:10.4.7-1.el7.centos                                   
  boost-program-options.x86_64 0:1.53.0-27.el7                                  
  galera-4.x86_64 0:26.4.2-1.rhel7.el7.centos                                   
  libaio.x86_64 0:0.3.109-13.el7                                                
  lsof.x86_64 0:4.87-6.el7                                                      
  perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7                                  
  perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7                                   
  perl-DBI.x86_64 0:1.627-4.el7                                                 
  perl-Data-Dumper.x86_64 0:2.145-3.el7                                         
  perl-IO-Compress.noarch 0:2.061-2.el7                                         
  perl-Net-Daemon.noarch 0:0.48-5.el7                                           
  perl-PlRPC.noarch 0:0.2020-14.el7                                             

Replaced:
  mariadb-libs.x86_64 1:5.5.60-1.el7_5                                          

Complete!
[root@wordpress ~]# 
~~~

### 启动 mysql

~~~
[root@wordpress ~]# systemctl start mariadb
[root@wordpress ~]# systemctl status mariadb
● mariadb.service - MariaDB 10.4.7 database server
   Loaded: loaded (/usr/lib/systemd/system/mariadb.service; enabled; vendor preset: disabled)
  Drop-In: /etc/systemd/system/mariadb.service.d
           └─migrated-from-my.cnf-settings.conf
   Active: active (running) since Sat 2019-08-24 02:16:43 UTC; 5s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
  Process: 10318 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
  Process: 10274 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`/usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-environment _WSREP_START_POSITION=$VAR || exit 1 (code=exited, status=0/SUCCESS)
  Process: 10272 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
 Main PID: 10286 (mysqld)
   Status: "Taking your SQL requests now..."
   CGroup: /system.slice/mariadb.service
           └─10286 /usr/sbin/mysqld

Aug 24 02:16:43 wordpress mysqld[10286]: 2019-08-24  2:16:43 0 [Note] InnoDB...1
Aug 24 02:16:43 wordpress mysqld[10286]: 2019-08-24  2:16:43 0 [Note] InnoDB...l
Aug 24 02:16:43 wordpress mysqld[10286]: 2019-08-24  2:16:43 0 [Note] InnoDB...3
Aug 24 02:16:43 wordpress mysqld[10286]: 2019-08-24  2:16:43 0 [Note] Plugin....
Aug 24 02:16:43 wordpress mysqld[10286]: 2019-08-24  2:16:43 0 [Note] Server....
Aug 24 02:16:43 wordpress mysqld[10286]: 2019-08-24  2:16:43 0 [Note] Readin...d
Aug 24 02:16:43 wordpress mysqld[10286]: 2019-08-24  2:16:43 0 [Note] Added ...e
Aug 24 02:16:43 wordpress mysqld[10286]: 2019-08-24  2:16:43 0 [Note] /usr/s....
Aug 24 02:16:43 wordpress mysqld[10286]: Version: '10.4.7-MariaDB'  socket: ...r
Aug 24 02:16:43 wordpress systemd[1]: Started MariaDB 10.4.7 database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@wordpress ~]# 
~~~


### 配置mysql

~~~
[root@wordpress ~]# mysql_secure_installation

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user. If you've just installed MariaDB, and
haven't set the root password yet, you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password or using the unix_socket ensures that nobody
can log into the MariaDB root user without the proper authorisation.

You already have your root account protected, so you can safely answer 'n'.

Switch to unix_socket authentication [Y/n] 
Enabled successfully!
Reloading privilege tables..
 ... Success!


You already have your root account protected, so you can safely answer 'n'.

Change the root password? [Y/n] n
 ... skipping.

By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] 
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] 
 ... Success!

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] 
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] 
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
[root@wordpress ~]# echo $?
0
[root@wordpress ~]# 
~~~

### 创建数据库和用户

~~~
[root@wordpress ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 16
Server version: 10.4.7-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.001 sec)

MariaDB [(none)]> CREATE DATABASE WP;
Query OK, 1 row affected (0.001 sec)

MariaDB [(none)]> GRANT ALL PRIVILEGES ON WP.* TO "wp"@"localhost" IDENTIFIED BY "wp";
Query OK, 0 rows affected (0.002 sec)

MariaDB [(none)]> FLUSH PRIVILEGES;
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> exit
Bye
[root@wordpress ~]# 
~~~

## 安装 Apache

~~~
[root@wordpress ~]# yum install  -y  httpd.x86_64 
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.cqu.edu.cn
 * updates: mirrors.huaweicloud.com
Resolving Dependencies
--> Running transaction check
---> Package httpd.x86_64 0:2.4.6-89.el7.centos.1 will be installed
--> Processing Dependency: httpd-tools = 2.4.6-89.el7.centos.1 for package: httpd-2.4.6-89.el7.centos.1.x86_64
--> Processing Dependency: system-logos >= 7.92.1-1 for package: httpd-2.4.6-89.el7.centos.1.x86_64
--> Processing Dependency: /etc/mime.types for package: httpd-2.4.6-89.el7.centos.1.x86_64
--> Processing Dependency: libaprutil-1.so.0()(64bit) for package: httpd-2.4.6-89.el7.centos.1.x86_64
--> Processing Dependency: libapr-1.so.0()(64bit) for package: httpd-2.4.6-89.el7.centos.1.x86_64
--> Running transaction check
---> Package apr.x86_64 0:1.4.8-3.el7_4.1 will be installed
---> Package apr-util.x86_64 0:1.5.2-6.el7 will be installed
---> Package centos-logos.noarch 0:70.0.6-3.el7.centos will be installed
---> Package httpd-tools.x86_64 0:2.4.6-89.el7.centos.1 will be installed
---> Package mailcap.noarch 0:2.1.41-2.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch        Version                       Repository    Size
================================================================================
Installing:
 httpd             x86_64      2.4.6-89.el7.centos.1         updates      2.7 M
Installing for dependencies:
 apr               x86_64      1.4.8-3.el7_4.1               base         103 k
 apr-util          x86_64      1.5.2-6.el7                   base          92 k
 centos-logos      noarch      70.0.6-3.el7.centos           base          21 M
 httpd-tools       x86_64      2.4.6-89.el7.centos.1         updates       91 k
 mailcap           noarch      2.1.41-2.el7                  base          31 k

Transaction Summary
================================================================================
Install  1 Package (+5 Dependent packages)

Total download size: 24 M
Installed size: 31 M
Downloading packages:
(1/6): apr-util-1.5.2-6.el7.x86_64.rpm                     |  92 kB   00:00     
(2/6): mailcap-2.1.41-2.el7.noarch.rpm                     |  31 kB   00:00     
(3/6): httpd-tools-2.4.6-89.el7.centos.1.x86_64.rpm        |  91 kB   00:00     
(4/6): httpd-2.4.6-89.el7.centos.1.x86_64.rpm              | 2.7 MB   00:00     
(5/6): apr-1.4.8-3.el7_4.1.x86_64.rpm                      | 103 kB   00:10     
(6/6): centos-logos-70.0.6-3.el7.centos.noarch.rpm         |  21 MB   00:27     
--------------------------------------------------------------------------------
Total                                              901 kB/s |  24 MB  00:27     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : apr-1.4.8-3.el7_4.1.x86_64                                   1/6 
  Installing : apr-util-1.5.2-6.el7.x86_64                                  2/6 
  Installing : httpd-tools-2.4.6-89.el7.centos.1.x86_64                     3/6 
  Installing : centos-logos-70.0.6-3.el7.centos.noarch                      4/6 
  Installing : mailcap-2.1.41-2.el7.noarch                                  5/6 
  Installing : httpd-2.4.6-89.el7.centos.1.x86_64                           6/6 
  Verifying  : httpd-2.4.6-89.el7.centos.1.x86_64                           1/6 
  Verifying  : httpd-tools-2.4.6-89.el7.centos.1.x86_64                     2/6 
  Verifying  : mailcap-2.1.41-2.el7.noarch                                  3/6 
  Verifying  : apr-util-1.5.2-6.el7.x86_64                                  4/6 
  Verifying  : apr-1.4.8-3.el7_4.1.x86_64                                   5/6 
  Verifying  : centos-logos-70.0.6-3.el7.centos.noarch                      6/6 

Installed:
  httpd.x86_64 0:2.4.6-89.el7.centos.1                                          

Dependency Installed:
  apr.x86_64 0:1.4.8-3.el7_4.1                                                  
  apr-util.x86_64 0:1.5.2-6.el7                                                 
  centos-logos.noarch 0:70.0.6-3.el7.centos                                     
  httpd-tools.x86_64 0:2.4.6-89.el7.centos.1                                    
  mailcap.noarch 0:2.1.41-2.el7                                                 

Complete!
[root@wordpress ~]# 
~~~

## 安装 php


因为系统自带的 php 版本太低，所以要使用其它的软件仓库来提供软件包

~~~
[root@wordpress ~]# rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
Retrieving http://rpms.remirepo.net/enterprise/remi-release-7.rpm
warning: /var/tmp/rpm-tmp.xVDdLk: Header V4 DSA/SHA1 Signature, key ID 00f97f56: NOKEY
error: Failed dependencies:
	epel-release = 7 is needed by remi-release-7.6-2.el7.remi.noarch
[root@wordpress ~]# yum -y install epel-release
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.cqu.edu.cn
 * updates: mirrors.huaweicloud.com
Resolving Dependencies
--> Running transaction check
---> Package epel-release.noarch 0:7-11 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                Arch             Version         Repository        Size
================================================================================
Installing:
 epel-release           noarch           7-11            extras            15 k

Transaction Summary
================================================================================
Install  1 Package

Total download size: 15 k
Installed size: 24 k
Downloading packages:
epel-release-7-11.noarch.rpm                               |  15 kB   00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : epel-release-7-11.noarch                                     1/1 
  Verifying  : epel-release-7-11.noarch                                     1/1 

Installed:
  epel-release.noarch 0:7-11                                                    

Complete!
[root@wordpress ~]# rpm -Uvh http://rpms.remirepo.net/enterprise/remi-release-7.rpm
Retrieving http://rpms.remirepo.net/enterprise/remi-release-7.rpm
warning: /var/tmp/rpm-tmp.wlbPV4: Header V4 DSA/SHA1 Signature, key ID 00f97f56: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:remi-release-7.6-2.el7.remi      ################################# [100%]
[root@wordpress ~]# rpm -ql remi-release
/etc/pki/rpm-gpg/RPM-GPG-KEY-remi
/etc/pki/rpm-gpg/RPM-GPG-KEY-remi2017
/etc/pki/rpm-gpg/RPM-GPG-KEY-remi2018
/etc/yum.repos.d/remi-glpi91.repo
/etc/yum.repos.d/remi-glpi92.repo
/etc/yum.repos.d/remi-glpi93.repo
/etc/yum.repos.d/remi-glpi94.repo
/etc/yum.repos.d/remi-modular.repo
/etc/yum.repos.d/remi-php54.repo
/etc/yum.repos.d/remi-php70.repo
/etc/yum.repos.d/remi-php71.repo
/etc/yum.repos.d/remi-php72.repo
/etc/yum.repos.d/remi-php73.repo
/etc/yum.repos.d/remi-safe.repo
/etc/yum.repos.d/remi.repo
[root@wordpress ~]# ll /etc/yum.repos.d/
total 92
-rw-r--r--. 1 root root 1664 Nov 23  2018 CentOS-Base.repo
-rw-r--r--. 1 root root 1309 Nov 23  2018 CentOS-CR.repo
-rw-r--r--. 1 root root  649 Nov 23  2018 CentOS-Debuginfo.repo
-rw-r--r--. 1 root root  314 Nov 23  2018 CentOS-fasttrack.repo
-rw-r--r--. 1 root root  630 Nov 23  2018 CentOS-Media.repo
-rw-r--r--. 1 root root 1331 Nov 23  2018 CentOS-Sources.repo
-rw-r--r--. 1 root root 5701 Nov 23  2018 CentOS-Vault.repo
-rw-r--r--. 1 root root  951 Oct  2  2017 epel.repo
-rw-r--r--. 1 root root 1050 Oct  2  2017 epel-testing.repo
-rw-r--r--. 1 root root  261 Aug 24 01:57 mariadb.repo
-rw-r--r--. 1 root root  446 Mar  8 07:34 remi-glpi91.repo
-rw-r--r--. 1 root root  446 Mar  8 07:34 remi-glpi92.repo
-rw-r--r--. 1 root root  446 Mar  8 07:34 remi-glpi93.repo
-rw-r--r--. 1 root root  446 Mar  8 07:34 remi-glpi94.repo
-rw-r--r--. 1 root root  855 Mar  8 07:34 remi-modular.repo
-rw-r--r--. 1 root root  456 Mar  8 07:34 remi-php54.repo
-rw-r--r--. 1 root root 1314 Mar  8 07:34 remi-php70.repo
-rw-r--r--. 1 root root 1314 Mar  8 07:34 remi-php71.repo
-rw-r--r--. 1 root root 1314 Mar  8 07:34 remi-php72.repo
-rw-r--r--. 1 root root 1314 Mar  8 07:34 remi-php73.repo
-rw-r--r--. 1 root root 2605 Mar  8 07:34 remi.repo
-rw-r--r--. 1 root root  750 Mar  8 07:34 remi-safe.repo
[root@wordpress ~]#
~~~

### 启用最新的 php 仓库

~~~
[root@wordpress ~]# yum-config-manager --enable remi-php73
Loaded plugins: fastestmirror
=============================== repo: remi-php73 ===============================
[remi-php73]
async = True
bandwidth = 0
base_persistdir = /var/lib/yum/repos/x86_64/7
baseurl = 
cache = 0
cachedir = /var/cache/yum/x86_64/7/remi-php73
check_config_file_age = True
compare_providers_priority = 80
cost = 1000
deltarpm_metadata_percentage = 100
deltarpm_percentage = 
enabled = 1
enablegroups = True
exclude = 
failovermethod = priority
ftp_disable_epsv = False
gpgcadir = /var/lib/yum/repos/x86_64/7/remi-php73/gpgcadir
gpgcakey = 
gpgcheck = True
gpgdir = /var/lib/yum/repos/x86_64/7/remi-php73/gpgdir
gpgkey = file:///etc/pki/rpm-gpg/RPM-GPG-KEY-remi
hdrdir = /var/cache/yum/x86_64/7/remi-php73/headers
http_caching = all
includepkgs = 
ip_resolve = 
keepalive = True
keepcache = False
mddownloadpolicy = sqlite
mdpolicy = group:small
mediaid = 
metadata_expire = 21600
metadata_expire_filter = read-only:present
metalink = 
minrate = 0
mirrorlist = http://cdn.remirepo.net/enterprise/7/php73/mirror
mirrorlist_expire = 86400
name = Remi's PHP 7.3 RPM repository for Enterprise Linux 7 - x86_64
old_base_cache_dir = 
password = 
persistdir = /var/lib/yum/repos/x86_64/7/remi-php73
pkgdir = /var/cache/yum/x86_64/7/remi-php73/packages
proxy = False
proxy_dict = 
proxy_password = 
proxy_username = 
repo_gpgcheck = False
retries = 10
skip_if_unavailable = False
ssl_check_cert_permissions = True
sslcacert = 
sslclientcert = 
sslclientkey = 
sslverify = True
throttle = 0
timeout = 30.0
ui_id = remi-php73
ui_repoid_vars = releasever,
   basearch
username = 

[root@wordpress ~]# 
~~~


### 安装 php

~~~
[root@wordpress ~]# yum -y install php php-opcache
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
epel/x86_64/metalink                                     | 4.1 kB     00:00     
 * base: mirrors.aliyun.com
 * epel: hkg.mirror.rackspace.com
 * extras: mirrors.cqu.edu.cn
 * remi-php73: mirror.innosol.asia
 * remi-safe: mirror.innosol.asia
 * updates: mirrors.huaweicloud.com
epel                                                     | 5.3 kB     00:00     
remi-php73                                               | 3.0 kB     00:00     
remi-safe                                                | 3.0 kB     00:00     
(1/5): remi-php73/primary_db                               | 202 kB   00:00     
(2/5): epel/x86_64/group_gz                                |  88 kB   00:00     
(3/5): remi-safe/primary_db                                | 1.6 MB   00:01     
(4/5): epel/x86_64/updateinfo                              | 1.0 MB   00:56     
(5/5): epel/x86_64/primary_db                              | 6.8 MB   06:35     
Resolving Dependencies
--> Running transaction check
---> Package php.x86_64 0:7.3.8-1.el7.remi will be installed
--> Processing Dependency: php-cli(x86-64) = 7.3.8-1.el7.remi for package: php-7.3.8-1.el7.remi.x86_64
--> Processing Dependency: php-common(x86-64) = 7.3.8-1.el7.remi for package: php-7.3.8-1.el7.remi.x86_64
--> Processing Dependency: libargon2.so.0()(64bit) for package: php-7.3.8-1.el7.remi.x86_64
---> Package php-opcache.x86_64 0:7.3.8-1.el7.remi will be installed
--> Running transaction check
---> Package libargon2.x86_64 0:20161029-3.el7 will be installed
---> Package php-cli.x86_64 0:7.3.8-1.el7.remi will be installed
---> Package php-common.x86_64 0:7.3.8-1.el7.remi will be installed
--> Processing Dependency: php-json(x86-64) = 7.3.8-1.el7.remi for package: php-common-7.3.8-1.el7.remi.x86_64
--> Running transaction check
---> Package php-json.x86_64 0:7.3.8-1.el7.remi will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch         Version                  Repository        Size
================================================================================
Installing:
 php               x86_64       7.3.8-1.el7.remi         remi-php73       3.2 M
 php-opcache       x86_64       7.3.8-1.el7.remi         remi-php73       308 k
Installing for dependencies:
 libargon2         x86_64       20161029-3.el7           epel              23 k
 php-cli           x86_64       7.3.8-1.el7.remi         remi-php73       4.9 M
 php-common        x86_64       7.3.8-1.el7.remi         remi-php73       1.1 M
 php-json          x86_64       7.3.8-1.el7.remi         remi-php73        65 k

Transaction Summary
================================================================================
Install  2 Packages (+4 Dependent packages)

Total download size: 9.6 M
Installed size: 38 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/epel/packages/libargon2-20161029-3.el7.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID 352c64e5: NOKEY
Public key for libargon2-20161029-3.el7.x86_64.rpm is not installed
(1/6): libargon2-20161029-3.el7.x86_64.rpm                 |  23 kB   00:00     
warning: /var/cache/yum/x86_64/7/remi-php73/packages/php-json-7.3.8-1.el7.remi.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 00f97f56: NOKEY
Public key for php-json-7.3.8-1.el7.remi.x86_64.rpm is not installed
(2/6): php-json-7.3.8-1.el7.remi.x86_64.rpm                |  65 kB   00:01     
(3/6): php-7.3.8-1.el7.remi.x86_64.rpm                     | 3.2 MB   00:02     
(4/6): php-common-7.3.8-1.el7.remi.x86_64.rpm              | 1.1 MB   00:04     
(5/6): php-opcache-7.3.8-1.el7.remi.x86_64.rpm             | 308 kB   00:07     
(6/6): php-cli-7.3.8-1.el7.remi.x86_64.rpm                 | 4.9 MB   00:53     
--------------------------------------------------------------------------------
Total                                              183 kB/s | 9.6 MB  00:53     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-remi
Importing GPG key 0x00F97F56:
 Userid     : "Remi Collet <RPMS@FamilleCollet.com>"
 Fingerprint: 1ee0 4cce 88a4 ae4a a29a 5df5 004e 6f47 00f9 7f56
 Package    : remi-release-7.6-2.el7.remi.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-remi
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Importing GPG key 0x352C64E5:
 Userid     : "Fedora EPEL (7) <epel@fedoraproject.org>"
 Fingerprint: 91e9 7d7c 4a5e 96f1 7f3e 888f 6a2f aea2 352c 64e5
 Package    : epel-release-7-11.noarch (@extras)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
Warning: RPMDB altered outside of yum.
  Installing : libargon2-20161029-3.el7.x86_64                              1/6 
  Installing : php-common-7.3.8-1.el7.remi.x86_64                           2/6 
  Installing : php-json-7.3.8-1.el7.remi.x86_64                             3/6 
  Installing : php-cli-7.3.8-1.el7.remi.x86_64                              4/6 
  Installing : php-7.3.8-1.el7.remi.x86_64                                  5/6 
  Installing : php-opcache-7.3.8-1.el7.remi.x86_64                          6/6 
  Verifying  : php-cli-7.3.8-1.el7.remi.x86_64                              1/6 
  Verifying  : php-7.3.8-1.el7.remi.x86_64                                  2/6 
  Verifying  : php-opcache-7.3.8-1.el7.remi.x86_64                          3/6 
  Verifying  : libargon2-20161029-3.el7.x86_64                              4/6 
  Verifying  : php-json-7.3.8-1.el7.remi.x86_64                             5/6 
  Verifying  : php-common-7.3.8-1.el7.remi.x86_64                           6/6 

Installed:
  php.x86_64 0:7.3.8-1.el7.remi      php-opcache.x86_64 0:7.3.8-1.el7.remi     

Dependency Installed:
  libargon2.x86_64 0:20161029-3.el7       php-cli.x86_64 0:7.3.8-1.el7.remi    
  php-common.x86_64 0:7.3.8-1.el7.remi    php-json.x86_64 0:7.3.8-1.el7.remi   

Complete!
[root@wordpress ~]# php
php      php-cgi  phpize   
[root@wordpress ~]# php --version
PHP 7.3.8 (cli) (built: Jul 30 2019 09:26:16) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.3.8, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.3.8, Copyright (c) 1999-2018, by Zend Technologies
[root@wordpress ~]# 
~~~


### 通过 web 进行测试　

~~~
[root@wordpress ~]# vim /var/www/html/info.php
[root@wordpress ~]# cat /var/www/html/info.php
<?php
phpinfo();
[root@wordpress ~]# 
[root@wordpress ~]# systemctl start httpd.service
[root@wordpress ~]# 
~~~


![wordpress](/assets/img/wordpress/wordpress10.png)

## 安装 php mysql 驱动

~~~
[root@wordpress ~]#  yum -y install php-mysqlnd php-pdo
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * epel: hkg.mirror.rackspace.com
 * extras: mirrors.cqu.edu.cn
 * remi-php73: mirror.innosol.asia
 * remi-safe: mirror.innosol.asia
 * updates: mirrors.huaweicloud.com
Resolving Dependencies
--> Running transaction check
---> Package php-mysqlnd.x86_64 0:7.3.8-1.el7.remi will be installed
---> Package php-pdo.x86_64 0:7.3.8-1.el7.remi will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package           Arch         Version                  Repository        Size
================================================================================
Installing:
 php-mysqlnd       x86_64       7.3.8-1.el7.remi         remi-php73       233 k
 php-pdo           x86_64       7.3.8-1.el7.remi         remi-php73       127 k

Transaction Summary
================================================================================
Install  2 Packages

Total download size: 360 k
Installed size: 1.2 M
Downloading packages:
(1/2): php-mysqlnd-7.3.8-1.el7.remi.x86_64.rpm             | 233 kB   00:00     
(2/2): php-pdo-7.3.8-1.el7.remi.x86_64.rpm                 | 127 kB   00:02     
--------------------------------------------------------------------------------
Total                                              170 kB/s | 360 kB  00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : php-pdo-7.3.8-1.el7.remi.x86_64                              1/2 
  Installing : php-mysqlnd-7.3.8-1.el7.remi.x86_64                          2/2 
  Verifying  : php-pdo-7.3.8-1.el7.remi.x86_64                              1/2 
  Verifying  : php-mysqlnd-7.3.8-1.el7.remi.x86_64                          2/2 

Installed:
  php-mysqlnd.x86_64 0:7.3.8-1.el7.remi    php-pdo.x86_64 0:7.3.8-1.el7.remi   

Complete!
[root@wordpress ~]#
[root@wordpress ~]# systemctl restart httpd.service
[root@wordpress ~]# 
~~~


![wordpress](/assets/img/wordpress/wordpress11.png)


## 安装 wordpress

### 下载软件包

~~~
[root@wordpress ~]# wget http://wordpress.org/latest.tar.gz
--2019-08-24 02:18:32--  http://wordpress.org/latest.tar.gz
Resolving wordpress.org (wordpress.org)... 198.143.164.252
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:80... connected.
HTTP request sent, awaiting response... 301 Moved Permanently
Location: https://wordpress.org/latest.tar.gz [following]
--2019-08-24 02:18:33--  https://wordpress.org/latest.tar.gz
Connecting to wordpress.org (wordpress.org)|198.143.164.252|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 11200591 (11M) [application/octet-stream]
Saving to: ‘latest.tar.gz’

100%[======================================>] 11,200,591   388KB/s   in 26s    

2019-08-24 02:19:01 (415 KB/s) - ‘latest.tar.gz’ saved [11200591/11200591]

[root@wordpress ~]# ll latest.tar.gz 
-rw-r--r--. 1 root root 11200591 Jun 18 17:52 latest.tar.gz
[root@wordpress ~]# du -sh latest.tar.gz 
11M	latest.tar.gz
[root@wordpress ~]# 
~~~


### 解压包

~~~
[root@wordpress ~]# ls
anaconda-ks.cfg  latest.tar.gz  original-ks.cfg
[root@wordpress ~]# tar -zxvf latest.tar.gz 
wordpress/
wordpress/xmlrpc.php
wordpress/wp-blog-header.php
wordpress/readme.html
wordpress/wp-signup.php
wordpress/index.php
wordpress/wp-cron.php
wordpress/wp-config-sample.php
wordpress/wp-login.php
wordpress/wp-settings.php
wordpress/license.txt
wordpress/wp-content/
．．．
．．．
wordpress/wp-admin/post-new.php
wordpress/wp-admin/themes.php
wordpress/wp-admin/options-reading.php
wordpress/wp-trackback.php
wordpress/wp-comments-post.php
[root@wordpress ~]# ls
anaconda-ks.cfg  latest.tar.gz  original-ks.cfg  wordpress
[root@wordpress ~]# 
~~~


### 拷贝到合适的位置并且修改权限

~~~
[root@wordpress ~]# cp -rp wordpress/ /var/www/html/
[root@wordpress ~]# chown -R apache.apache /var/www/html/wordpress/
[root@wordpress ~]# ll /var/www/html/
total 8
-rw-r--r--. 1 root   root     17 Aug 24 03:40 info.php
drwxr-xr-x. 5 apache apache 4096 Jun 18 17:50 wordpress
[root@wordpress ~]# 
~~~


## 初始化配置

访问 **`http://10.1.0.200/wordpress/wp-admin/setup-config.php`**

这个配置过程的结果就是生成一个 **wp-config.php** 文件

登录后第一个界面

![wordpress](/assets/img/wordpress/wordpress12.png)

选择语言

![wordpress](/assets/img/wordpress/wordpress13.png)

提示需要连接数据库的相关信息

![wordpress](/assets/img/wordpress/wordpress14.png)

把数据库的信息填入

![wordpress](/assets/img/wordpress/wordpress15.png)

连接正常后会提示开始安装实例

![wordpress](/assets/img/wordpress/wordpress16.png)

关于实例的配置信息

![wordpress](/assets/img/wordpress/wordpress18.png)

命名并且创建用户

![wordpress](/assets/img/wordpress/wordpress19.png)

提示创建成功

![wordpress](/assets/img/wordpress/wordpress20.png)

进行登录

![wordpress](/assets/img/wordpress/wordpress21.png)

进入 **wordpress** 系统

![wordpress](/assets/img/wordpress/wordpress22.png)


这样 **WordPress** 就安装成功了

---

# 总结

WordPress 是一个经典的 LAMP 应用 

所以在 Linux Apache Mysql PHP 等环境准备好了的情况下只需要将相应的软件包解压并放在合适的位置就可以了

不得不说当前的 wordpress 版本颜值极高，插件支持，媒体库的托拽，编辑窗口的易用，内容版本差异的管理，评论支持，主题更换，界面的自定义都有着十分友好的接口

一个开源内容发布系统，真的很喜欢

* TOC
{:toc}

---

[wordpress]:https://wordpress.org/
[wordpress_dep]:https://wordpress.org/about/requirements/
[wordpress_install]:https://codex.wordpress.org/Installing_WordPress
[mariadb_repo]:https://downloads.mariadb.org/mariadb/repositories/#distro=CentOS&distro_release=centos7-amd64--centos7&mirror=liteserver&version=10.4
