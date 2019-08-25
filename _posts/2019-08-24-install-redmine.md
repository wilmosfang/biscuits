---
layout: post
title: "Install Redmine"
date: 2019-08-24 15:45:59
image: '/assets/img/'
description: 'Redmine 的安装方法'
main-class: redmine
color: 
tags:
 - redmine
 - postgresql
 - ruby
 - rails
categories:
 - redmine
twitter_text: 'simple process of redmine installation'
introduction: 'installation method of redmine'
---


# 前言


**[Redmine][redmine]** 是一个灵活的项目管理软件，是在 **Ruby on Rails** 的 web 框架上构建的

>Redmine is a flexible project management web application written using Ruby on Rails framework.

主要具备以下功能

~~~
● 多项目和子项目支持
● 里程碑版本跟踪
● 可配置的用户角色控制
● 可配置的问题追踪系统
● 自动日历和甘特图绘制
● 支持 Blog 形式的新闻发布、Wiki 形式的文档撰写和文件管理
● RSS 输出和邮件通知
● 每个项目可以配置独立的 Wiki 和论坛模块
● 简单的任务时间跟踪机制
● 用户、项目、问题支持自定义属性
● 支持多 LDAP 用户认证
● 支持用户自注册和用户激活
● 多语言支持（已经内置了zh简体中文）
● 多数据库支持（MySQL、SQLite、PostgreSQL）
● 外观模版化定制（可以使用 Basecamp 、Ruby安装）
~~~


同类软件有:

* Trac: Python
* Jira+Confluence: Java
* ActiveCollab: PHP


在事务繁多的情况下，使用项目管理工具来对接请求服务，跟踪问题，管理事务，协调人力是一种有效的组织方法

这里分享一下 **[Redmine][redmine]** 的安装方法

参考 **[Installing Redmine][redmine_install]**

> **Tip:** 当前版本 **Redmine 4.0.4**



## 依赖

### ruby

* Ruby 2.2 (2.2.2 and later), 2.3, 2.4, 2.5, 2.6
* Rails 5.2

### 数据库

* MySQL 5.5 - 5.7
* PostgreSQL 9.2 or higher
* SQLite 3 (not for multi-user production use!)
* Microsoft SQL Server 2012 or higher



---



# 操作

## 系统环境

~~~
[root@redmine ~]# hostnamectl 
   Static hostname: redmine
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 0f1e88ce11bc4fc4ac8880d248b830cb
           Boot ID: dd2f7973d0ae48c8874241fa0d4d8391
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@redmine ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 83106sec preferred_lft 83106sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:b4:19:ed brd ff:ff:ff:ff:ff:ff
    inet 10.1.0.201/24 brd 10.1.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feb4:19ed/64 scope link 
       valid_lft forever preferred_lft forever
[root@redmine ~]# 
~~~

## 安装gcc

~~~
[root@redmine ~]# yum install gcc
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.cn99.com
 * updates: mirrors.cn99.com
Package gcc-4.8.5-36.el7_6.2.x86_64 already installed and latest version
Nothing to do
[root@redmine ~]# 
~~~

## 安装 rvm

~~~
[root@redmine ~]#  curl -L get.rvm.io | bash -s stable
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   194  100   194    0     0    290      0 --:--:-- --:--:-- --:--:--   290
100 24535  100 24535    0     0   1380      0  0:00:17  0:00:17 --:--:--  5367
Downloading https://github.com/rvm/rvm/archive/1.29.9.tar.gz
Downloading https://github.com/rvm/rvm/releases/download/1.29.9/1.29.9.tar.gz.asc
curl: (28) Connection timed out after 30002 milliseconds

Could not download 'https://github.com/rvm/rvm/releases/download/1.29.9/1.29.9.tar.gz.asc'.
  curl returned status '28'.

Creating group 'rvm'
Installing RVM to /usr/local/rvm/
Installation of RVM in /usr/local/rvm/ is almost complete:

  * First you need to add all users that will be using rvm to 'rvm' group,
    and logout - login again, anyone using rvm will be operating with `umask u=rwx,g=rwx,o=rx`.

  * To start using RVM you need to run `source /etc/profile.d/rvm.sh`
    in all your open shell windows, in rare cases you need to reopen all shell windows.
  * Please do NOT forget to add your users to the <code>rvm</code> group.
     The installer no longer auto-adds root or users to the rvm group. Admins must do this.
     Also, please note that group memberships are ONLY evaluated at login time.
     This means that users must log out then back in before group membership takes effect!
<warn>Thanks for installing RVM 🙏</warn>
Please consider donating to our open collective to help us maintain RVM.

👉  Donate: <code>https://opencollective.com/rvm/donate</code>


[root@redmine ~]# echo $?
0
[root@redmine ~]# 
~~~

## 安装 ruby

~~~
[root@redmine ~]# rvm install ruby
Searching for binary rubies, this might take some time.
No binary rubies available for: centos/7/x86_64/ruby-2.6.3.
Continuing with compilation. Please read 'rvm help mount' to get more information on binary rubies.
Checking requirements for centos.
Installing requirements for centos.
Installing required packages: patch, autoconf, automake, bison, gcc-c++, libffi-devel, libtool, patch, readline-devel, ruby, sqlite-devel, zlib-devel, openssl-devel..........................
Requirements installation successful.
Installing Ruby from source to: /usr/local/rvm/rubies/ruby-2.6.3, this may take a while depending on your cpu(s)...
ruby-2.6.3 - #downloading ruby-2.6.3, this may take a while depending on your connection...
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 13.8M  100 13.8M    0     0  10013      0  0:24:09  0:24:08  0:00:01 18063
ruby-2.6.3 - #extracting ruby-2.6.3 to /usr/local/rvm/src/ruby-2.6.3.....
ruby-2.6.3 - #configuring......................................................................
ruby-2.6.3 - #post-configuration..
ruby-2.6.3 - #compiling................................................................................................
ruby-2.6.3 - #installing................................
ruby-2.6.3 - #making binaries executable..
ruby-2.6.3 - #downloading rubygems-3.0.6
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  866k  100  866k    0     0   8246      0  0:01:47  0:01:47 --:--:--  7813
No checksum for downloaded archive, recording checksum in user configuration.
ruby-2.6.3 - #extracting rubygems-3.0.6.....
ruby-2.6.3 - #removing old rubygems........
ruby-2.6.3 - #installing rubygems-3.0.6...............................................
ruby-2.6.3 - #gemset created /usr/local/rvm/gems/ruby-2.6.3@global
ruby-2.6.3 - #importing gemset /usr/local/rvm/gemsets/global.gems................................................................
ruby-2.6.3 - #generating global wrappers.......
ruby-2.6.3 - #gemset created /usr/local/rvm/gems/ruby-2.6.3
ruby-2.6.3 - #importing gemsetfile /usr/local/rvm/gemsets/default.gems evaluated to empty gem list
ruby-2.6.3 - #generating default wrappers.......
ruby-2.6.3 - #adjusting #shebangs for (gem irb erb ri rdoc testrb rake).
Install of ruby-2.6.3 - #complete 
Ruby was built without documentation, to build it run: rvm docs generate-ri
[root@redmine ~]# echo $?
0
[root@redmine ~]#
~~~

## 安装 postgresql

先安装库

~~~
[root@redmine ~]#  yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm -y
Loaded plugins: fastestmirror
pgdg-redhat-repo-latest.noarch.rpm                                                     | 5.6 kB  00:00:00     
Examining /var/tmp/yum-root-3kCg_l/pgdg-redhat-repo-latest.noarch.rpm: pgdg-redhat-repo-42.0-4.noarch
Marking /var/tmp/yum-root-3kCg_l/pgdg-redhat-repo-latest.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package pgdg-redhat-repo.noarch 0:42.0-4 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================
 Package                    Arch             Version          Repository                                 Size
==============================================================================================================
Installing:
 pgdg-redhat-repo           noarch           42.0-4           /pgdg-redhat-repo-latest.noarch           6.8 k

Transaction Summary
==============================================================================================================
Install  1 Package

Total size: 6.8 k
Installed size: 6.8 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : pgdg-redhat-repo-42.0-4.noarch                                                             1/1 
  Verifying  : pgdg-redhat-repo-42.0-4.noarch             rv                                                1/1 

Installed:
  pgdg-redhat-repo.noarch 0:42.0-4                                                                            

Complete!
[root@redmine ~]# 
~~~

进行软件安装

~~~
[root@redmine ~]# yum install postgresql11 postgresql11-server -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.cn99.com
 * updates: mirrors.cn99.com
Resolving Dependencies
--> Running transaction check
---> Package postgresql11.x86_64 0:11.5-1PGDG.rhel7 will be installed
--> Processing Dependency: postgresql11-libs(x86-64) = 11.5-1PGDG.rhel7 for package: postgresql11-11.5-1PGDG.rhel7.x86_64
--> Processing Dependency: libpq.so.5()(64bit) for package: postgresql11-11.5-1PGDG.rhel7.x86_64
---> Package postgresql11-server.x86_64 0:11.5-1PGDG.rhel7 will be installed
--> Running transaction check
---> Package postgresql11-libs.x86_64 0:11.5-1PGDG.rhel7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

==============================================================================================================
 Package                          Arch                Version                       Repository           Size
==============================================================================================================
Installing:
 postgresql11                     x86_64              11.5-1PGDG.rhel7              pgdg11              1.6 M
 postgresql11-server              x86_64              11.5-1PGDG.rhel7              pgdg11              4.7 M
Installing for dependencies:
 postgresql11-libs                x86_64              11.5-1PGDG.rhel7              pgdg11              361 k

Transaction Summary
==============================================================================================================
Install  2 Packages (+1 Dependent package)

Total download size: 6.7 M
Installed size: 29 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/pgdg11/packages/postgresql11-libs-11.5-1PGDG.rhel7.x86_64.rpm: Header V4 DSA/SHA1 Signature, key ID 442df0f8: NOKEY
Public key for postgresql11-libs-11.5-1PGDG.rhel7.x86_64.rpm is not installed
(1/3): postgresql11-libs-11.5-1PGDG.rhel7.x86_64.rpm                                   | 361 kB  00:00:50     
(2/3): postgresql11-11.5-1PGDG.rhel7.x86_64.rpm                                         | 1.6 MB  00:03:13     
(3/3): postgresql11-server-11.5-1PGDG.rhel7.x86_64.rpm                                  | 4.7 MB  00:08:46     
---------------------------------------------------------------------------------------------------------------
Total                                                                           12 kB/s | 6.7 MB  00:09:36     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG
Importing GPG key 0x442DF0F8:
 Userid     : "PostgreSQL RPM Building Project <pgsqlrpms-hackers@pgfoundry.org>"
 Fingerprint: 68c9 e2b9 1a37 d136 fe74 d176 1f16 d2e1 442d f0f8
 Package    : pgdg-redhat-repo-42.0-4.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : postgresql11-libs-11.5-1PGDG.rhel7.x86_64                                                   1/3 
  Installing : postgresql11-11.5-1PGDG.rhel7.x86_64                                                        2/3 
  Installing : postgresql11-server-11.5-1PGDG.rhel7.x86_64                                                 3/3 
  Verifying  : postgresql11-libs-11.5-1PGDG.rhel7.x86_64                                                   1/3 
  Verifying  : postgresql11-server-11.5-1PGDG.rhel7.x86_64                                                 2/3 
  Verifying  : postgresql11-11.5-1PGDG.rhel7.x86_64                                                        3/3 

Installed:
  postgresql11.x86_64 0:11.5-1PGDG.rhel7             postgresql11-server.x86_64 0:11.5-1PGDG.rhel7            

Dependency Installed:
  postgresql11-libs.x86_64 0:11.5-1PGDG.rhel7                                                                  

Complete!
[root@redmine ~]
~~~

初始化数据库

~~~
[root@redmine ~]# /usr/pgsql-11/bin/postgresql-11-setup initdb
Initializing database ... OK

[root@redmine ~]# 
~~~

启动数据库

~~~
[root@redmine ~]# systemctl enable postgresql-11
Created symlink from /etc/systemd/system/multi-user.target.wants/postgresql-11.service to /usr/lib/systemd/system/postgresql-11.service.
[root@redmine ~]# systemctl status  postgresql-11
● postgresql-11.service - PostgreSQL 11 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-11.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: https://www.postgresql.org/docs/11/static/
[root@redmine ~]#  systemctl start   postgresql-11
[root@redmine ~]# systemctl status  postgresql-11
● postgresql-11.service - PostgreSQL 11 database server
   Loaded: loaded (/usr/lib/systemd/system/postgresql-11.service; enabled; vendor preset: disabled)
   Active: active (running) since Sat 2019-08-24 17:26:36 UTC; 9s ago
     Docs: https://www.postgresql.org/docs/11/static/
  Process: 2642 ExecStartPre=/usr/pgsql-11/bin/postgresql-11-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
 Main PID: 2647 (postmaster)
   CGroup: /system.slice/postgresql-11.service
           ├─2647 /usr/pgsql-11/bin/postmaster -D /var/lib/pgsql/11/data/
           ├─2649 postgres: logger   
           ├─2651 postgres: checkpointer   
           ├─2652 postgres: background writer   
           ├─2653 postgres: walwriter   
           ├─2654 postgres: autovacuum launcher   
           ├─2655 postgres: stats collector   
           └─2656 postgres: logical replication launcher   

Aug 24 17:26:36 redmine systemd[1]: Starting PostgreSQL 11 database server...
Aug 24 17:26:36 redmine postmaster[2647]: 2019-08-24 17:26:36.193 UTC [2647] LOG:  listening on IPv6 ad...5432
Aug 24 17:26:36 redmine postmaster[2647]: 2019-08-24 17:26:36.193 UTC [2647] LOG:  listening on IPv4 ad...5432
Aug 24 17:26:36 redmine postmaster[2647]: 2019-08-24 17:26:36.196 UTC [2647] LOG:  listening on Unix so...432"
Aug 24 17:26:36 redmine postmaster[2647]: 2019-08-24 17:26:36.198 UTC [2647] LOG:  listening on Unix so...432"
Aug 24 17:26:36 redmine postmaster[2647]: 2019-08-24 17:26:36.208 UTC [2647] LOG:  redirecting log outp...cess
Aug 24 17:26:36 redmine postmaster[2647]: 2019-08-24 17:26:36.208 UTC [2647] HINT:  Future log output w...og".
Aug 24 17:26:36 redmine systemd[1]: Started PostgreSQL 11 database server.
Hint: Some lines were ellipsized, use -l to show in full.
[root@redmine ~]# 
~~~

进行访问

~~~
[root@redmine ~]# su - postgres
[root@redmine ~]# 
[root@redmine ~]# 
-bash-4.2$ 
-bash-4.2$ psql
psql (11.5)
Type "help" for help.

postgres=# select version();
                                                 version                                                 
---------------------------------------------------------------------------------------------------------
 PostgreSQL 11.5 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (Red Hat 4.8.5-36), 64-bit
(1 row)

postgres=#
postgres=# \q
-bash-4.2$
~~~


## 创建数据库和用户

~~~
postgres=# CREATE ROLE redmine LOGIN ENCRYPTED PASSWORD 'redmine' NOINHERIT VALID UNTIL 'infinity';
CREATE ROLE
postgres=# CREATE DATABASE redmine WITH ENCODING='UTF8' OWNER=redmine;
CREATE DATABASE
postgres=# \db
       List of tablespaces
    Name    |  Owner   | Location 
------------|----------|----------
 pg_default | postgres | 
 pg_global  | postgres | 
(2 rows)

postgres=# \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------|----------|----------|-------------|-------------|-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 redmine   | redmine  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)

postgres=# 
~~~

创建对应的系统用户

~~~
[root@redmine redmine-4.0.4]# useradd redmine
[root@redmine redmine-4.0.4]# passwd redmine
Changing password for user redmine.
New password: 
BAD PASSWORD: The password is shorter than 8 characters
Retype new password: 
passwd: all authentication tokens updated successfully.
[root@redmine redmine-4.0.4]# su - redmine
[redmine@redmine ~]$ psql
psql (9.2.24, server 11.5)
WARNING: psql version 9.2, server version 11.0.
         Some psql features might not work.
Type "help" for help.

redmine=> \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------|----------|----------|-------------|-------------|-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 redmine   | redmine  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)

redmine=> 
~~~

## 配置 pg_hba.conf

~~~
[root@redmine redmine-4.0.4]# vim /var/lib/pgsql/11/data/pg_hba.conf 
[root@redmine redmine-4.0.4]# grep -v "#" /var/lib/pgsql/11/data/pg_hba.conf  | cat -s

host   all             all             127.0.0.1/32            md5
local   all             all                                     peer
host    all             all             127.0.0.1/32            ident
host    all             all             ::1/128                 ident
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            ident
host    replication     all             ::1/128                 ident
[root@redmine redmine-4.0.4]# 
~~~

>**Note:** 如果不配置会有如下报错

~~~
[root@redmine redmine-4.0.4]# psql -h 127.0.0.1 -U redmine -d redmine 
psql: FATAL:  Ident authentication failed for user "redmine"
[root@redmine redmine-4.0.4]#
~~~

后面的 **ROR** 连接数据库也会有相同的问题

要用用户密码登录测试，确保可以通过网络进行登录

~~~
[root@redmine redmine-4.0.4]# psql -h 127.0.0.1 -U redmine -d redmine 
Password for user redmine: 
psql (9.2.24, server 11.5)
WARNING: psql version 9.2, server version 11.0.
         Some psql features might not work.
Type "help" for help.

redmine=> \l
                                  List of databases
   Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
-----------|----------|----------|-------------|-------------|-----------------------
 postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 redmine   | redmine  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
 template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
           |          |          |             |             | postgres=CTc/postgres
(4 rows)

redmine=> \q
[root@redmine redmine-4.0.4]# 
~~~

## 下载 Redmine

~~~
[root@redmine ~]#  wget https://www.redmine.org/releases/redmine-4.0.4.tar.gz
--2019-08-24 16:28:28--  https://www.redmine.org/releases/redmine-4.0.4.tar.gz
Resolving www.redmine.org (www.redmine.org)... 46.4.36.71
Connecting to www.redmine.org (www.redmine.org)|46.4.36.71|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2519330 (2.4M) [application/x-gzip]
Saving to: ‘redmine-4.0.4.tar.gz’

100%[====================================================================>] 2,519,330   10.0KB/s   in 4m 31s 

2019-08-24 16:33:02 (9.06 KB/s) - ‘redmine-4.0.4.tar.gz’ saved [2519330/2519330]

[root@redmine ~]# 
~~~

## 解压 redmine

~~~
[root@redmine ~]# tar -zxvf redmine-4.0.4.tar.gz
redmine-4.0.4/
redmine-4.0.4/app/
redmine-4.0.4/app/models/
redmine-4.0.4/app/models/document.rb
redmine-4.0.4/app/models/user_custom_field.rb
redmine-4.0.4/app/models/comment.rb
redmine-4.0.4/app/models/issue_custom_field.rb
redmine-4.0.4/app/models/issue_import.rb
redmine-4.0.4/app/models/group_anonymous.rb
redmine-4.0.4/app/models/change.rb
...
...
redmine-4.0.4/bin/
redmine-4.0.4/bin/about
redmine-4.0.4/bin/changelog.rb
redmine-4.0.4/bin/rails
redmine-4.0.4/bin/bundle
redmine-4.0.4/bin/rake
[root@redmine ~]
~~~


## 配置数据库

~~~
[root@redmine config]# pwd
/root/redmine-4.0.4/config
[root@redmine config]# cp database.yml.example  database.yml 
[root@redmine config]# vim  database.yml
[root@redmine config]# grep -v "#" database.yml | cat -s 

production:
  adapter: postgresql
  database: redmine
  host: 127.0.0.1
  username: redmine
  password: "redmine"

[root@redmine config]# 
~~~

从这个配置文件中 **Redmine** 会自动读取和安装相应的数据库驱动

如果对这个配置进行了修改，需要重新运行一次 **`bundle install --without development test`**

## 安装 bundler

~~~
[root@redmine redmine-4.0.4]# gem install bundler
Fetching bundler-2.0.2.gem
Successfully installed bundler-2.0.2
Parsing documentation for bundler-2.0.2
Installing ri documentation for bundler-2.0.2
Done installing documentation for bundler after 2 seconds
1 gem installed
[root@redmine redmine-4.0.4]# 
~~~


## 安装 pg 包的依赖

~~~
[root@redmine redmine-4.0.4]#  yum install postgresql-devel -y
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * extras: mirrors.cn99.com
 * updates: mirrors.cn99.com
Resolving Dependencies
--> Running transaction check
---> Package postgresql-devel.x86_64 0:9.2.24-1.el7_5 will be installed
--> Processing Dependency: postgresql-libs(x86-64) = 9.2.24-1.el7_5 for package: postgresql-devel-9.2.24-1.el7_5.x86_64
--> Processing Dependency: postgresql(x86-64) = 9.2.24-1.el7_5 for package: postgresql-devel-9.2.24-1.el7_5.x86_64
--> Running transaction check
---> Package postgresql.x86_64 0:9.2.24-1.el7_5 will be installed
---> Package postgresql-libs.x86_64 0:9.2.24-1.el7_5 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                 Arch          Version                Repository   Size
================================================================================
Installing:
 postgresql-devel        x86_64        9.2.24-1.el7_5         base        952 k
Installing for dependencies:
 postgresql              x86_64        9.2.24-1.el7_5         base        3.0 M
 postgresql-libs         x86_64        9.2.24-1.el7_5         base        234 k

Transaction Summary
================================================================================
Install  1 Package (+2 Dependent packages)

Total download size: 4.2 M
Installed size: 21 M
Downloading packages:
(1/3): postgresql-9.2.24-1.el7_5.x86_64.rpm                | 3.0 MB   00:02     
(2/3): postgresql-libs-9.2.24-1.el7_5.x86_64.rpm           | 234 kB   00:05     
(3/3): postgresql-devel-9.2.24-1.el7_5.x86_64.rpm          | 952 kB   00:08     
--------------------------------------------------------------------------------
Total                                              510 kB/s | 4.2 MB  00:08     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : postgresql-libs-9.2.24-1.el7_5.x86_64                        1/3 
  Installing : postgresql-9.2.24-1.el7_5.x86_64                             2/3 
  Installing : postgresql-devel-9.2.24-1.el7_5.x86_64                       3/3 
  Verifying  : postgresql-devel-9.2.24-1.el7_5.x86_64                       1/3 
  Verifying  : postgresql-libs-9.2.24-1.el7_5.x86_64                        2/3 
  Verifying  : postgresql-9.2.24-1.el7_5.x86_64                             3/3 

Installed:
  postgresql-devel.x86_64 0:9.2.24-1.el7_5                                      

Dependency Installed:
  postgresql.x86_64 0:9.2.24-1.el7_5   postgresql-libs.x86_64 0:9.2.24-1.el7_5  

Complete!
[root@redmine redmine-4.0.4]# gem install pg
Building native extensions. This could take a while...
Successfully installed pg-1.1.4
Parsing documentation for pg-1.1.4
Installing ri documentation for pg-1.1.4
Done installing documentation for pg after 1 seconds
1 gem installed
[root@redmine redmine-4.0.4]# 
~~~

>**Note:** 如果不安装此开发包，在 gp gem 的安装过程中，会有报错

~~~
...
...
Installing pg 1.1.4 with native extensions
Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

    current directory: /usr/local/rvm/gems/ruby-2.6.3/gems/pg-1.1.4/ext
/usr/local/rvm/rubies/ruby-2.6.3/bin/ruby -I
/usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/site_ruby/2.6.0 -r
./siteconf20190825-24088-1wqzt43.rb extconf.rb
checking for pg_config... no
No pg_config... trying anyway. If building fails, please try again with
 --with-pg-config=/path/to/pg_config
checking for libpq-fe.h... no
Can't find the 'libpq-fe.h header
*** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.

Provided configuration options:
	--with-opt-dir
	--without-opt-dir
	--with-opt-include
	--without-opt-include=${opt-dir}/include
	--with-opt-lib
	--without-opt-lib=${opt-dir}/lib
	--with-make-prog
	--without-make-prog
	--srcdir=.
	--curdir
	--ruby=/usr/local/rvm/rubies/ruby-2.6.3/bin/$(RUBY_BASE_NAME)
	--with-pg
	--without-pg
	--enable-windows-cross
	--disable-windows-cross
	--with-pg-config
	--without-pg-config
	--with-pg_config
	--without-pg_config
	--with-pg-dir
	--without-pg-dir
	--with-pg-include
	--without-pg-include=${pg-dir}/include
	--with-pg-lib
	--without-pg-lib=${pg-dir}/lib

To see why this extension failed to compile, please check the mkmf.log which can
be found here:

  /usr/local/rvm/gems/ruby-2.6.3/extensions/x86_64-linux/2.6.0/pg-1.1.4/mkmf.log

extconf failed, exit code 1

Gem files will remain installed in /usr/local/rvm/gems/ruby-2.6.3/gems/pg-1.1.4
for inspection.
Results logged to
/usr/local/rvm/gems/ruby-2.6.3/extensions/x86_64-linux/2.6.0/pg-1.1.4/gem_make.out

An error occurred while installing pg (1.1.4), and Bundler cannot
continue.
Make sure that `gem install pg -v '1.1.4' --source
'https://gems.ruby-china.com/'` succeeds before bundling.

In Gemfile:
  pg
[root@redmine redmine-4.0.4]# gem install pg -v '1.1.4' --source
ERROR:  While executing gem ... (OptionParser::MissingArgument)
    missing argument: --source
[root@redmine redmine-4.0.4]# gem install pg -v '1.1.4' 
Building native extensions. This could take a while...
ERROR:  Error installing pg:
	ERROR: Failed to build gem native extension.

    current directory: /usr/local/rvm/gems/ruby-2.6.3/gems/pg-1.1.4/ext
/usr/local/rvm/rubies/ruby-2.6.3/bin/ruby -I /usr/local/rvm/rubies/ruby-2.6.3/lib/ruby/site_ruby/2.6.0 -r ./siteconf20190825-1747-1d3vclw.rb extconf.rb
checking for pg_config... no
No pg_config... trying anyway. If building fails, please try again with
 --with-pg-config=/path/to/pg_config
checking for libpq-fe.h... no
Can't find the 'libpq-fe.h header
*** extconf.rb failed ***
Could not create Makefile due to some reason, probably lack of necessary
libraries and/or headers.  Check the mkmf.log file for more details.  You may
need configuration options.

Provided configuration options:
	--with-opt-dir
	--without-opt-dir
	--with-opt-include
	--without-opt-include=${opt-dir}/include
	--with-opt-lib
	--without-opt-lib=${opt-dir}/lib
	--with-make-prog
	--without-make-prog
	--srcdir=.
	--curdir
	--ruby=/usr/local/rvm/rubies/ruby-2.6.3/bin/$(RUBY_BASE_NAME)
	--with-pg
	--without-pg
	--enable-windows-cross
	--disable-windows-cross
	--with-pg-config
	--without-pg-config
	--with-pg_config
	--without-pg_config
	--with-pg-dir
	--without-pg-dir
	--with-pg-include
	--without-pg-include=${pg-dir}/include
	--with-pg-lib
	--without-pg-lib=${pg-dir}/lib

To see why this extension failed to compile, please check the mkmf.log which can be found here:

  /usr/local/rvm/gems/ruby-2.6.3/extensions/x86_64-linux/2.6.0/pg-1.1.4/mkmf.log

extconf failed, exit code 1

Gem files will remain installed in /usr/local/rvm/gems/ruby-2.6.3/gems/pg-1.1.4 for inspection.
Results logged to /usr/local/rvm/gems/ruby-2.6.3/extensions/x86_64-linux/2.6.0/pg-1.1.4/gem_make.out
[root@redmine redmine-4.0.4]# echo $?
1
[root@redmine redmine-4.0.4]#
~~~


## bundler install 

~~~
[root@redmine redmine-4.0.4]# pwd
/root/redmine-4.0.4
[root@redmine redmine-4.0.4]#
[root@redmine redmine-4.0.4]# bundle install --without development test
Don't run Bundler as root. Bundler can ask for sudo if it is needed, and
installing your bundle as root will break this application for all non-root
users on this machine.
The dependency tzinfo-data (>= 0) will be unused by any of the platforms Bundler is installing for. Bundler is installing for ruby but the dependency is only for x86-mingw32, x64-mingw32, x86-mswin32. To add those platforms to the bundle, run `bundle lock --add-platform x86-mingw32 x64-mingw32 x86-mswin32`.
Fetching gem metadata from https://gems.ruby-china.com/.............
Fetching gem metadata from https://gems.ruby-china.com/.
Resolving dependencies...
Using rake 12.3.3
Using concurrent-ruby 1.1.5
Using i18n 0.7.0
Using minitest 5.11.3
Using thread_safe 0.3.6
Using tzinfo 1.2.5
Using activesupport 5.2.3
Using builder 3.2.3
Using erubi 1.8.0
Using mini_portile2 2.4.0
Using nokogiri 1.10.4
Using rails-dom-testing 2.0.3
Using crass 1.0.4
Using loofah 2.2.3
Using rails-html-sanitizer 1.2.0
Using actionview 5.2.3
Using rack 2.0.7
Using rack-test 1.1.0
Using actionpack 5.2.3
Using nio4r 2.4.0
Using websocket-extensions 0.1.4
Using websocket-driver 0.7.1
Using actioncable 5.2.3
Using globalid 0.4.2
Using activejob 5.2.3
Using mini_mime 1.0.2
Using mail 2.7.1
Using actionmailer 5.2.3
Using method_source 0.9.2
Using thor 0.20.3
Using railties 5.2.3
Using actionpack-xml_parser 2.0.1
Using activemodel 5.2.3
Using arel 9.0.0
Using activerecord 5.2.3
Using mimemagic 0.3.3
Using marcel 0.3.3
Using activestorage 5.2.3
Using public_suffix 3.1.1
Using addressable 2.6.0
Using bundler 2.0.2
Using css_parser 1.7.0
Using htmlentities 4.3.4
Using net-ldap 0.16.1
Using pg 1.1.4
Fetching ruby-openid 2.3.0
Installing ruby-openid 2.3.0
Fetching rack-openid 1.4.2
Installing rack-openid 1.4.2
Fetching sprockets 3.7.2
Installing sprockets 3.7.2
Fetching sprockets-rails 3.2.1
Installing sprockets-rails 3.2.1
Fetching rails 5.2.3
Installing rails 5.2.3
Fetching rbpdf-font 1.19.1
Installing rbpdf-font 1.19.1
Fetching rbpdf 1.19.8
Installing rbpdf 1.19.8
Fetching redcarpet 3.4.0
Installing redcarpet 3.4.0 with native extensions
Fetching request_store 1.0.5
Installing request_store 1.0.5
Fetching roadie 3.5.0
Installing roadie 3.5.0
Fetching roadie-rails 1.3.0
Installing roadie-rails 1.3.0
Fetching rouge 3.3.0
Installing rouge 3.3.0
Bundle complete! 26 Gemfile dependencies, 57 gems now installed.
Gems in the groups development, test and rmagick were not installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
[root@redmine redmine-4.0.4]# echo $?
0
[root@redmine redmine-4.0.4]# 
~~~


## 生成会话存储的密钥

~~~
[root@redmine redmine-4.0.4]# bundle exec rake generate_secret_token
[root@redmine redmine-4.0.4]# echo $?
0
[root@redmine redmine-4.0.4]# 
~~~


## 生成数据库结构


通过 rake migrate 来生成数据库结构

**`bundle exec rake db:migrate`**

~~~
[root@redmine redmine-4.0.4]# RAILS_ENV=production bundle exec rake db:migrate
== 1 Setup: migrating =========================================================
-- create_table("attachments", {:force=>true, :id=>:integer})
   -> 0.0079s
-- create_table("auth_sources", {:force=>true, :id=>:integer})
   -> 0.0057s
-- create_table("custom_fields", {:force=>true, :id=>:integer})
   -> 0.0083s
-- create_table("custom_fields_projects", {:id=>false, :force=>true})
   -> 0.0016s
-- create_table("custom_fields_trackers", {:id=>false, :force=>true})
   -> 0.0015s
-- create_table("custom_values", {:force=>true, :id=>:integer})
   -> 0.0047s
-- create_table("documents", {:force=>true, :id=>:integer})
   -> 0.0049s
-- add_index("documents", ["project_id"], {:name=>"documents_project_id"})
   -> 0.0043s
-- create_table("enumerations", {:force=>true, :id=>:integer})
   -> 0.0035s
-- create_table("issue_categories", {:force=>true, :id=>:integer})
   -> 0.0033s
-- add_index("issue_categories", ["project_id"], {:name=>"issue_categories_project_id"})
   -> 0.0028s
-- create_table("issue_histories", {:force=>true, :id=>:integer})
   -> 0.0045s
-- add_index("issue_histories", ["issue_id"], {:name=>"issue_histories_issue_id"})
   -> 0.0024s
-- create_table("issue_statuses", {:force=>true, :id=>:integer})
   -> 0.0036s
-- create_table("issues", {:force=>true, :id=>:integer})
   -> 0.0054s
-- add_index("issues", ["project_id"], {:name=>"issues_project_id"})
   -> 0.0024s
-- create_table("members", {:force=>true, :id=>:integer})
   -> 0.0035s
-- create_table("news", {:force=>true, :id=>:integer})
   -> 0.0046s
-- add_index("news", ["project_id"], {:name=>"news_project_id"})
   -> 0.0026s
-- create_table("permissions", {:force=>true, :id=>:integer})
   -> 0.0042s
-- create_table("permissions_roles", {:id=>false, :force=>true})
   -> 0.0015s
-- add_index("permissions_roles", ["role_id"], {:name=>"permissions_roles_role_id"})
   -> 0.0024s
-- create_table("projects", {:force=>true, :id=>:integer})
   -> 0.0049s
-- create_table("roles", {:force=>true, :id=>:integer})
   -> 0.0036s
-- create_table("tokens", {:force=>true, :id=>:integer})
   -> 0.0033s
-- create_table("trackers", {:force=>true, :id=>:integer})
   -> 0.0029s
-- create_table("users", {:force=>true, :id=>:integer})
   -> 0.0049s
-- create_table("versions", {:force=>true, :id=>:integer})
   -> 0.0044s
-- add_index("versions", ["project_id"], {:name=>"versions_project_id"})
   -> 0.0023s
-- create_table("workflows", {:force=>true, :id=>:integer})
   -> 0.0034s
== 1 Setup: migrated (0.1644s) ================================================

== 2 IssueMove: migrating =====================================================
== 2 IssueMove: migrated (0.0067s) ============================================

== 3 IssueAddNote: migrating ==================================================
== 3 IssueAddNote: migrated (0.0065s) =========================================

== 4 ExportPdf: migrating =====================================================
== 4 ExportPdf: migrated (0.0072s) ============================================

== 5 IssueStartDate: migrating ================================================
-- add_column(:issues, :start_date, :date, {})
   -> 0.0007s
-- add_column(:issues, :done_ratio, :integer, {:default=>0, :null=>false})
   -> 0.0009s
== 5 IssueStartDate: migrated (0.0017s) =======================================

== 6 CalendarAndActivity: migrating ===========================================
== 6 CalendarAndActivity: migrated (0.0076s) ==================================

== 7 CreateJournals: migrating ================================================
-- create_table(:journals, {:force=>true, :id=>:integer})
   -> 0.0045s
-- create_table(:journal_details, {:force=>true, :id=>:integer})
   -> 0.0044s
-- add_index("journals", ["journalized_id", "journalized_type"], {:name=>"journals_journalized_id"})
   -> 0.0026s
-- add_index("journal_details", ["journal_id"], {:name=>"journal_details_journal_id"})
   -> 0.0024s
-- drop_table(:issue_histories)
   -> 0.0025s
== 7 CreateJournals: migrated (0.0252s) =======================================

== 8 CreateUserPreferences: migrating =========================================
-- create_table(:user_preferences, {:id=>:integer})
   -> 0.0038s
== 8 CreateUserPreferences: migrated (0.0038s) ================================

== 9 AddHideMailPref: migrating ===============================================
-- add_column(:user_preferences, :hide_mail, :boolean, {:default=>false})
   -> 0.0010s
== 9 AddHideMailPref: migrated (0.0010s) ======================================

== 10 CreateComments: migrating ===============================================
-- create_table(:comments, {:id=>:integer})
   -> 0.0044s
== 10 CreateComments: migrated (0.0044s) ======================================

== 11 AddNewsCommentsCount: migrating =========================================
-- add_column(:news, :comments_count, :integer, {:default=>0, :null=>false})
   -> 0.0010s
== 11 AddNewsCommentsCount: migrated (0.0010s) ================================

== 12 AddCommentsPermissions: migrating =======================================
== 12 AddCommentsPermissions: migrated (0.0071s) ==============================

== 13 CreateQueries: migrating ================================================
-- create_table(:queries, {:force=>true, :id=>:integer})
   -> 0.0046s
== 13 CreateQueries: migrated (0.0047s) =======================================

== 14 AddQueriesPermissions: migrating ========================================
== 14 AddQueriesPermissions: migrated (0.0064s) ===============================

== 15 CreateRepositories: migrating ===========================================
-- create_table(:repositories, {:force=>true, :id=>:integer})
   -> 0.0041s
== 15 CreateRepositories: migrated (0.0042s) ==================================

== 16 AddRepositoriesPermissions: migrating ===================================
== 16 AddRepositoriesPermissions: migrated (0.0095s) ==========================

== 17 CreateSettings: migrating ===============================================
-- create_table(:settings, {:force=>true, :id=>:integer})
   -> 0.0041s
== 17 CreateSettings: migrated (0.0042s) ======================================

== 18 SetDocAndFilesNotifications: migrating ==================================
== 18 SetDocAndFilesNotifications: migrated (0.0121s) =========================

== 19 AddIssueStatusPosition: migrating =======================================
-- add_column(:issue_statuses, :position, :integer, {:default=>1})
   -> 0.0011s
== 19 AddIssueStatusPosition: migrated (0.0066s) ==============================

== 20 AddRolePosition: migrating ==============================================
-- add_column(:roles, :position, :integer, {:default=>1})
   -> 0.0009s
== 20 AddRolePosition: migrated (0.0272s) =====================================

== 21 AddTrackerPosition: migrating ===========================================
-- add_column(:trackers, :position, :integer, {:default=>1})
   -> 0.0010s
== 21 AddTrackerPosition: migrated (0.0115s) ==================================

== 22 SerializePossiblesValues: migrating =====================================
== 22 SerializePossiblesValues: migrated (0.0027s) ============================

== 23 AddTrackerIsInRoadmap: migrating ========================================
-- add_column(:trackers, :is_in_roadmap, :boolean, {:default=>true, :null=>false})
   -> 0.0011s
== 23 AddTrackerIsInRoadmap: migrated (0.0011s) ===============================

== 24 AddRoadmapPermission: migrating =========================================
== 24 AddRoadmapPermission: migrated (0.0064s) ================================

== 25 AddSearchPermission: migrating ==========================================
== 25 AddSearchPermission: migrated (0.0065s) =================================

== 26 AddRepositoryLoginAndPassword: migrating ================================
-- add_column(:repositories, :login, :string, {:limit=>60, :default=>""})
   -> 0.0010s
-- add_column(:repositories, :password, :string, {:limit=>60, :default=>""})
   -> 0.0008s
== 26 AddRepositoryLoginAndPassword: migrated (0.0019s) =======================

== 27 CreateWikis: migrating ==================================================
-- create_table(:wikis, {:id=>:integer})
   -> 0.0029s
-- add_index(:wikis, :project_id, {:name=>:wikis_project_id})
   -> 0.0024s
== 27 CreateWikis: migrated (0.0054s) =========================================

== 28 CreateWikiPages: migrating ==============================================
-- create_table(:wiki_pages, {:id=>:integer})
   -> 0.0025s
-- add_index(:wiki_pages, [:wiki_id, :title], {:name=>:wiki_pages_wiki_id_title})
   -> 0.0025s
== 28 CreateWikiPages: migrated (0.0052s) =====================================

== 29 CreateWikiContents: migrating ===========================================
-- create_table(:wiki_contents, {:id=>:integer})
   -> 0.0038s
-- add_index(:wiki_contents, :page_id, {:name=>:wiki_contents_page_id})
   -> 0.0025s
-- create_table(:wiki_content_versions, {:id=>:integer})
   -> 0.0041s
-- add_index(:wiki_content_versions, :wiki_content_id, {:name=>:wiki_content_versions_wcid})
   -> 0.0025s
== 29 CreateWikiContents: migrated (0.0131s) ==================================

== 30 AddProjectsFeedsPermissions: migrating ==================================
== 30 AddProjectsFeedsPermissions: migrated (0.0064s) =========================

== 31 AddRepositoryRootUrl: migrating =========================================
-- add_column(:repositories, :root_url, :string, {:limit=>255, :default=>""})
   -> 0.0010s
== 31 AddRepositoryRootUrl: migrated (0.0011s) ================================

== 32 CreateTimeEntries: migrating ============================================
-- create_table(:time_entries, {:id=>:integer})
   -> 0.0026s
-- add_index(:time_entries, [:project_id], {:name=>:time_entries_project_id})
   -> 0.0024s
-- add_index(:time_entries, [:issue_id], {:name=>:time_entries_issue_id})
   -> 0.0025s
== 32 CreateTimeEntries: migrated (0.0078s) ===================================

== 33 AddTimelogPermissions: migrating ========================================
== 33 AddTimelogPermissions: migrated (0.0064s) ===============================

== 34 CreateChangesets: migrating =============================================
-- create_table(:changesets, {:id=>:integer})
   -> 0.0038s
-- add_index(:changesets, [:repository_id, :revision], {:unique=>true, :name=>:changesets_repos_rev})
   -> 0.0025s
== 34 CreateChangesets: migrated (0.0064s) ====================================

== 35 CreateChanges: migrating ================================================
-- create_table(:changes, {:id=>:integer})
   -> 0.0041s
-- add_index(:changes, [:changeset_id], {:name=>:changesets_changeset_id})
   -> 0.0027s
== 35 CreateChanges: migrated (0.0070s) =======================================

== 36 AddChangesetCommitDate: migrating =======================================
-- add_column(:changesets, :commit_date, :date, {})
   -> 0.0007s
== 36 AddChangesetCommitDate: migrated (0.0185s) ==============================

== 37 AddProjectIdentifier: migrating =========================================
-- add_column(:projects, :identifier, :string, {:limit=>20})
   -> 0.0007s
== 37 AddProjectIdentifier: migrated (0.0007s) ================================

== 38 AddCustomFieldIsFilter: migrating =======================================
-- add_column(:custom_fields, :is_filter, :boolean, {:null=>false, :default=>false})
   -> 0.0009s
== 38 AddCustomFieldIsFilter: migrated (0.0010s) ==============================

== 39 CreateWatchers: migrating ===============================================
-- create_table(:watchers, {:id=>:integer})
   -> 0.0044s
== 39 CreateWatchers: migrated (0.0045s) ======================================

== 40 CreateChangesetsIssues: migrating =======================================
-- create_table(:changesets_issues, {:id=>false})
   -> 0.0009s
-- add_index(:changesets_issues, [:changeset_id, :issue_id], {:unique=>true, :name=>:changesets_issues_ids})
   -> 0.0025s
== 40 CreateChangesetsIssues: migrated (0.0036s) ==============================

== 41 RenameCommentToComments: migrating ======================================
== 41 RenameCommentToComments: migrated (0.0596s) =============================

== 42 CreateIssueRelations: migrating =========================================
-- create_table(:issue_relations, {:id=>:integer})
   -> 0.0045s
== 42 CreateIssueRelations: migrated (0.0046s) ================================

== 43 AddRelationsPermissions: migrating ======================================
== 43 AddRelationsPermissions: migrated (0.0076s) =============================

== 44 SetLanguageLengthToFive: migrating ======================================
-- change_column(:users, :language, :string, {:limit=>5})
   -> 0.0008s
== 44 SetLanguageLengthToFive: migrated (0.0029s) =============================

== 45 CreateBoards: migrating =================================================
-- create_table(:boards, {:id=>:integer})
   -> 0.0045s
-- add_index(:boards, [:project_id], {:name=>:boards_project_id})
   -> 0.0025s
== 45 CreateBoards: migrated (0.0071s) ========================================

== 46 CreateMessages: migrating ===============================================
-- create_table(:messages, {:id=>:integer})
   -> 0.0043s
-- add_index(:messages, [:board_id], {:name=>:messages_board_id})
   -> 0.0025s
-- add_index(:messages, [:parent_id], {:name=>:messages_parent_id})
   -> 0.0024s
== 46 CreateMessages: migrated (0.0095s) ======================================

== 47 AddBoardsPermissions: migrating =========================================
== 47 AddBoardsPermissions: migrated (0.0079s) ================================

== 48 AllowNullVersionEffectiveDate: migrating ================================
-- change_column(:versions, :effective_date, :date, {})
   -> 0.0008s
== 48 AllowNullVersionEffectiveDate: migrated (0.0027s) =======================

== 49 AddWikiDestroyPagePermission: migrating =================================
== 49 AddWikiDestroyPagePermission: migrated (0.0063s) ========================

== 50 AddWikiAttachmentsPermissions: migrating ================================
== 50 AddWikiAttachmentsPermissions: migrated (0.0071s) =======================

== 51 AddProjectStatus: migrating =============================================
-- add_column(:projects, :status, :integer, {:default=>1, :null=>false})
   -> 0.0012s
== 51 AddProjectStatus: migrated (0.0012s) ====================================

== 52 AddChangesRevision: migrating ===========================================
-- add_column(:changes, :revision, :string, {})
   -> 0.0005s
== 52 AddChangesRevision: migrated (0.0007s) ==================================

== 53 AddChangesBranch: migrating =============================================
-- add_column(:changes, :branch, :string, {})
   -> 0.0008s
== 53 AddChangesBranch: migrated (0.0009s) ====================================

== 54 AddChangesetsScmid: migrating ===========================================
-- add_column(:changesets, :scmid, :string, {})
   -> 0.0007s
== 54 AddChangesetsScmid: migrated (0.0008s) ==================================

== 55 AddRepositoriesType: migrating ==========================================
-- add_column(:repositories, :type, :string, {})
   -> 0.0006s
== 55 AddRepositoriesType: migrated (0.0036s) =================================

== 56 AddRepositoriesChangesPermission: migrating =============================
== 56 AddRepositoriesChangesPermission: migrated (0.0245s) ====================

== 57 AddVersionsWikiPageTitle: migrating =====================================
-- add_column(:versions, :wiki_page_title, :string, {})
   -> 0.0006s
== 57 AddVersionsWikiPageTitle: migrated (0.0008s) ============================

== 58 AddIssueCategoriesAssignedToId: migrating ===============================
-- add_column(:issue_categories, :assigned_to_id, :integer, {})
   -> 0.0005s
== 58 AddIssueCategoriesAssignedToId: migrated (0.0007s) ======================

== 59 AddRolesAssignable: migrating ===========================================
-- add_column(:roles, :assignable, :boolean, {:default=>true})
   -> 0.0010s
== 59 AddRolesAssignable: migrated (0.0010s) ==================================

== 60 ChangeChangesetsCommitterLimit: migrating ===============================
-- change_column(:changesets, :committer, :string, {:limit=>nil})
   -> 0.0006s
== 60 ChangeChangesetsCommitterLimit: migrated (0.0007s) ======================

== 61 AddRolesBuiltin: migrating ==============================================
-- add_column(:roles, :builtin, :integer, {:default=>0, :null=>false})
   -> 0.0010s
== 61 AddRolesBuiltin: migrated (0.0011s) =====================================

== 62 InsertBuiltinRoles: migrating ===========================================
== 62 InsertBuiltinRoles: migrated (0.0333s) ==================================

== 63 AddRolesPermissions: migrating ==========================================
-- add_column(:roles, :permissions, :text, {})
   -> 0.0021s
== 63 AddRolesPermissions: migrated (0.0022s) =================================

== 64 DropPermissions: migrating ==============================================
-- drop_table(:permissions)
   -> 0.0011s
-- drop_table(:permissions_roles)
   -> 0.0007s
== 64 DropPermissions: migrated (0.0018s) =====================================

== 65 AddSettingsUpdatedOn: migrating =========================================
-- add_column(:settings, :updated_on, :timestamp, {})
   -> 0.0007s
== 65 AddSettingsUpdatedOn: migrated (0.0128s) ================================

== 66 AddCustomValueCustomizedIndex: migrating ================================
-- add_index(:custom_values, [:customized_type, :customized_id], {:name=>:custom_values_customized})
   -> 0.0026s
== 66 AddCustomValueCustomizedIndex: migrated (0.0027s) =======================

== 67 CreateWikiRedirects: migrating ==========================================
-- create_table(:wiki_redirects, {:id=>:integer})
   -> 0.0039s
-- add_index(:wiki_redirects, [:wiki_id, :title], {:name=>:wiki_redirects_wiki_id_title})
   -> 0.0025s
== 67 CreateWikiRedirects: migrated (0.0066s) =================================

== 68 CreateEnabledModules: migrating =========================================
-- create_table(:enabled_modules, {:id=>:integer})
   -> 0.0037s
-- add_index(:enabled_modules, [:project_id], {:name=>:enabled_modules_project_id})
   -> 0.0025s
== 68 CreateEnabledModules: migrated (0.0089s) ================================

== 69 AddIssuesEstimatedHours: migrating ======================================
-- add_column(:issues, :estimated_hours, :float, {})
   -> 0.0006s
== 69 AddIssuesEstimatedHours: migrated (0.0007s) =============================

== 70 ChangeAttachmentsContentTypeLimit: migrating ============================
-- change_column(:attachments, :content_type, :string, {:limit=>nil})
   -> 0.0007s
== 70 ChangeAttachmentsContentTypeLimit: migrated (0.0008s) ===================

== 71 AddQueriesColumnNames: migrating ========================================
-- add_column(:queries, :column_names, :text, {})
   -> 0.0006s
== 71 AddQueriesColumnNames: migrated (0.0007s) ===============================

== 72 AddEnumerationsPosition: migrating ======================================
-- add_column(:enumerations, :position, :integer, {:default=>1})
   -> 0.0010s
== 72 AddEnumerationsPosition: migrated (0.0135s) =============================

== 73 AddEnumerationsIsDefault: migrating =====================================
-- add_column(:enumerations, :is_default, :boolean, {:default=>false, :null=>false})
   -> 0.0010s
== 73 AddEnumerationsIsDefault: migrated (0.0011s) ============================

== 74 AddAuthSourcesTls: migrating ============================================
-- add_column(:auth_sources, :tls, :boolean, {:default=>false, :null=>false})
   -> 0.0010s
== 74 AddAuthSourcesTls: migrated (0.0011s) ===================================

== 75 AddMembersMailNotification: migrating ===================================
-- add_column(:members, :mail_notification, :boolean, {:default=>false, :null=>false})
   -> 0.0009s
== 75 AddMembersMailNotification: migrated (0.0010s) ==========================

== 76 AllowNullPosition: migrating ============================================
-- change_column(:issue_statuses, :position, :integer, {})
   -> 0.0008s
-- change_column(:roles, :position, :integer, {})
   -> 0.0007s
-- change_column(:trackers, :position, :integer, {})
   -> 0.0008s
-- change_column(:boards, :position, :integer, {})
   -> 0.0007s
-- change_column(:enumerations, :position, :integer, {})
   -> 0.0008s
== 76 AllowNullPosition: migrated (0.0139s) ===================================

== 77 RemoveIssueStatusesHtmlColor: migrating =================================
-- remove_column(:issue_statuses, :html_color)
   -> 0.0006s
== 77 RemoveIssueStatusesHtmlColor: migrated (0.0007s) ========================

== 78 AddCustomFieldsPosition: migrating ======================================
-- add_column(:custom_fields, :position, :integer, {:default=>1})
   -> 0.0010s
== 78 AddCustomFieldsPosition: migrated (0.0018s) =============================

== 79 AddUserPreferencesTimeZone: migrating ===================================
-- add_column(:user_preferences, :time_zone, :string, {})
   -> 0.0006s
== 79 AddUserPreferencesTimeZone: migrated (0.0007s) ==========================

== 80 AddUsersType: migrating =================================================
-- add_column(:users, :type, :string, {})
   -> 0.0020s
== 80 AddUsersType: migrated (0.0274s) ========================================

== 81 CreateProjectsTrackers: migrating =======================================
-- create_table(:projects_trackers, {:id=>false})
   -> 0.0015s
-- add_index(:projects_trackers, :project_id, {:name=>:projects_trackers_project_id})
   -> 0.0025s
== 81 CreateProjectsTrackers: migrated (0.0054s) ==============================

== 82 AddMessagesLocked: migrating ============================================
-- add_column(:messages, :locked, :boolean, {:default=>false})
   -> 0.0009s
== 82 AddMessagesLocked: migrated (0.0010s) ===================================

== 83 AddMessagesSticky: migrating ============================================
-- add_column(:messages, :sticky, :integer, {:default=>0})
   -> 0.0011s
== 83 AddMessagesSticky: migrated (0.0012s) ===================================

== 84 ChangeAuthSourcesAccountLimit: migrating ================================
-- change_column(:auth_sources, :account, :string, {:limit=>nil})
   -> 0.0006s
== 84 ChangeAuthSourcesAccountLimit: migrated (0.0007s) =======================

== 85 AddRoleTrackerOldStatusIndexToWorkflows: migrating ======================
-- add_index(:workflows, [:role_id, :tracker_id, :old_status_id], {:name=>:wkfs_role_tracker_old_status})
   -> 0.0025s
== 85 AddRoleTrackerOldStatusIndexToWorkflows: migrated (0.0026s) =============

== 86 AddCustomFieldsSearchable: migrating ====================================
-- add_column(:custom_fields, :searchable, :boolean, {:default=>false})
   -> 0.0009s
== 86 AddCustomFieldsSearchable: migrated (0.0010s) ===========================

== 87 ChangeProjectsDescriptionToText: migrating ==============================
-- change_column(:projects, :description, :text, {})
   -> 0.0007s
== 87 ChangeProjectsDescriptionToText: migrated (0.0029s) =====================

== 88 AddCustomFieldsDefaultValue: migrating ==================================
-- add_column(:custom_fields, :default_value, :text, {})
   -> 0.0006s
== 88 AddCustomFieldsDefaultValue: migrated (0.0007s) =========================

== 89 AddAttachmentsDescription: migrating ====================================
-- add_column(:attachments, :description, :string, {})
   -> 0.0005s
== 89 AddAttachmentsDescription: migrated (0.0007s) ===========================

== 90 ChangeVersionsNameLimit: migrating ======================================
-- change_column(:versions, :name, :string, {:limit=>nil})
   -> 0.0007s
== 90 ChangeVersionsNameLimit: migrated (0.0008s) =============================

== 91 ChangeChangesetsRevisionToString: migrating =============================
-- index_exists?(:changesets, [:repository_id, :revision], {:name=>"changesets_repos_rev"})
   -> 0.0023s
-- remove_index(:changesets, {:name=>:changesets_repos_rev})
   -> 0.0005s
-- index_exists?(:changesets, [:repository_id, :revision], {:name=>"altered_changesets_repos_rev"})
   -> 0.0008s
-- change_column(:changesets, :revision, :string, {})
   -> 0.0034s
-- add_index(:changesets, [:repository_id, :revision], {:unique=>true, :name=>:changesets_repos_rev})
   -> 0.0024s
== 91 ChangeChangesetsRevisionToString: migrated (0.0112s) ====================

== 92 ChangeChangesFromRevisionToString: migrating ============================
-- change_column(:changes, :from_revision, :string, {})
   -> 0.0041s
== 92 ChangeChangesFromRevisionToString: migrated (0.0042s) ===================

== 93 AddWikiPagesProtected: migrating ========================================
-- add_column(:wiki_pages, :protected, :boolean, {:default=>false, :null=>false})
   -> 0.0010s
== 93 AddWikiPagesProtected: migrated (0.0010s) ===============================

== 94 ChangeProjectsHomepageLimit: migrating ==================================
-- change_column(:projects, :homepage, :string, {:limit=>nil})
   -> 0.0008s
== 94 ChangeProjectsHomepageLimit: migrated (0.0026s) =========================

== 95 AddWikiPagesParentId: migrating =========================================
-- add_column(:wiki_pages, :parent_id, :integer, {:default=>nil})
   -> 0.0008s
== 95 AddWikiPagesParentId: migrated (0.0009s) ================================

== 96 AddCommitAccessPermission: migrating ====================================
== 96 AddCommitAccessPermission: migrated (0.0010s) ===========================

== 97 AddViewWikiEditsPermission: migrating ===================================
== 97 AddViewWikiEditsPermission: migrated (0.0008s) ==========================

== 98 SetTopicAuthorsAsWatchers: migrating ====================================
== 98 SetTopicAuthorsAsWatchers: migrated (0.0147s) ===========================

== 99 AddDeleteWikiPagesAttachmentsPermission: migrating ======================
== 99 AddDeleteWikiPagesAttachmentsPermission: migrated (0.0008s) =============

== 100 AddChangesetsUserId: migrating =========================================
-- add_column(:changesets, :user_id, :integer, {:default=>nil})
   -> 0.0008s
== 100 AddChangesetsUserId: migrated (0.0009s) ================================

== 101 PopulateChangesetsUserId: migrating ====================================
== 101 PopulateChangesetsUserId: migrated (0.0009s) ===========================

== 102 AddCustomFieldsEditable: migrating =====================================
-- add_column(:custom_fields, :editable, :boolean, {:default=>true})
   -> 0.0010s
== 102 AddCustomFieldsEditable: migrated (0.0011s) ============================

== 103 SetCustomFieldsEditable: migrating =====================================
== 103 SetCustomFieldsEditable: migrated (0.0038s) ============================

== 104 AddProjectsLftAndRgt: migrating ========================================
-- add_column(:projects, :lft, :integer, {})
   -> 0.0006s
-- add_column(:projects, :rgt, :integer, {})
   -> 0.0005s
== 104 AddProjectsLftAndRgt: migrated (0.0012s) ===============================

== 105 BuildProjectsTree: migrating ===========================================
== 105 BuildProjectsTree: migrated (0.0037s) ==================================

== 106 RemoveProjectsProjectsCount: migrating =================================
-- remove_column(:projects, :projects_count)
   -> 0.0006s
== 106 RemoveProjectsProjectsCount: migrated (0.0007s) ========================

== 107 AddOpenIdAuthenticationTables: migrating ===============================
-- create_table(:open_id_authentication_associations, {:force=>true, :id=>:integer})
   -> 0.0040s
-- create_table(:open_id_authentication_nonces, {:force=>true, :id=>:integer})
   -> 0.0038s
== 107 AddOpenIdAuthenticationTables: migrated (0.0081s) ======================

== 108 AddIdentityUrlToUsers: migrating =======================================
-- add_column(:users, :identity_url, :string, {})
   -> 0.0006s
== 108 AddIdentityUrlToUsers: migrated (0.0007s) ==============================

== 20090214190337 AddWatchersUserIdTypeIndex: migrating =======================
-- add_index(:watchers, [:user_id, :watchable_type], {:name=>:watchers_user_id_type})
   -> 0.0027s
== 20090214190337 AddWatchersUserIdTypeIndex: migrated (0.0027s) ==============

== 20090312172426 AddQueriesSortCriteria: migrating ===========================
-- add_column(:queries, :sort_criteria, :text, {})
   -> 0.0006s
== 20090312172426 AddQueriesSortCriteria: migrated (0.0007s) ==================

== 20090312194159 AddProjectsTrackersUniqueIndex: migrating ===================
-- add_index(:projects_trackers, [:project_id, :tracker_id], {:name=>:projects_trackers_unique, :unique=>true})
   -> 0.0026s
== 20090312194159 AddProjectsTrackersUniqueIndex: migrated (0.0037s) ==========

== 20090318181151 ExtendSettingsName: migrating ===============================
-- change_column(:settings, :name, :string, {:limit=>255})
   -> 0.0008s
== 20090318181151 ExtendSettingsName: migrated (0.0040s) ======================

== 20090323224724 AddTypeToEnumerations: migrating ============================
-- add_column(:enumerations, :type, :string, {})
   -> 0.0019s
== 20090323224724 AddTypeToEnumerations: migrated (0.0020s) ===================

== 20090401221305 UpdateEnumerationsToSti: migrating ==========================
== 20090401221305 UpdateEnumerationsToSti: migrated (0.0057s) =================

== 20090401231134 AddActiveFieldToEnumerations: migrating =====================
-- add_column(:enumerations, :active, :boolean, {:default=>true, :null=>false})
   -> 0.0010s
== 20090401231134 AddActiveFieldToEnumerations: migrated (0.0011s) ============

== 20090403001910 AddProjectToEnumerations: migrating =========================
-- add_column(:enumerations, :project_id, :integer, {:null=>true, :default=>nil})
   -> 0.0010s
-- add_index(:enumerations, :project_id)
   -> 0.0026s
== 20090403001910 AddProjectToEnumerations: migrated (0.0037s) ================

== 20090406161854 AddParentIdToEnumerations: migrating ========================
-- add_column(:enumerations, :parent_id, :integer, {:null=>true, :default=>nil})
   -> 0.0009s
== 20090406161854 AddParentIdToEnumerations: migrated (0.0010s) ===============

== 20090425161243 AddQueriesGroupBy: migrating ================================
-- add_column(:queries, :group_by, :string, {})
   -> 0.0005s
== 20090425161243 AddQueriesGroupBy: migrated (0.0006s) =======================

== 20090503121501 CreateMemberRoles: migrating ================================
-- create_table(:member_roles, {:id=>:integer})
   -> 0.0024s
== 20090503121501 CreateMemberRoles: migrated (0.0024s) =======================

== 20090503121505 PopulateMemberRoles: migrating ==============================
== 20090503121505 PopulateMemberRoles: migrated (0.0088s) =====================

== 20090503121510 DropMembersRoleId: migrating ================================
-- remove_column(:members, :role_id)
   -> 0.0006s
== 20090503121510 DropMembersRoleId: migrated (0.0007s) =======================

== 20090614091200 FixMessagesStickyNull: migrating ============================
== 20090614091200 FixMessagesStickyNull: migrated (0.0030s) ===================

== 20090704172350 PopulateUsersType: migrating ================================
== 20090704172350 PopulateUsersType: migrated (0.0009s) =======================

== 20090704172355 CreateGroupsUsers: migrating ================================
-- create_table(:groups_users, {:id=>false})
   -> 0.0008s
-- add_index(:groups_users, [:group_id, :user_id], {:unique=>true, :name=>:groups_users_ids})
   -> 0.0028s
== 20090704172355 CreateGroupsUsers: migrated (0.0039s) =======================

== 20090704172358 AddMemberRolesInheritedFrom: migrating ======================
-- add_column(:member_roles, :inherited_from, :integer, {})
   -> 0.0006s
== 20090704172358 AddMemberRolesInheritedFrom: migrated (0.0006s) =============

== 20091010093521 FixUsersCustomValues: migrating =============================
== 20091010093521 FixUsersCustomValues: migrated (0.0042s) ====================

== 20091017212227 AddMissingIndexesToWorkflows: migrating =====================
-- add_index(:workflows, :old_status_id)
   -> 0.0026s
-- add_index(:workflows, :role_id)
   -> 0.0023s
-- add_index(:workflows, :new_status_id)
   -> 0.0024s
== 20091017212227 AddMissingIndexesToWorkflows: migrated (0.0076s) ============

== 20091017212457 AddMissingIndexesToCustomFieldsProjects: migrating ==========
-- add_index(:custom_fields_projects, [:custom_field_id, :project_id])
   -> 0.0026s
== 20091017212457 AddMissingIndexesToCustomFieldsProjects: migrated (0.0027s) =

== 20091017212644 AddMissingIndexesToMessages: migrating ======================
-- add_index(:messages, :last_reply_id)
   -> 0.0025s
-- add_index(:messages, :author_id)
   -> 0.0023s
== 20091017212644 AddMissingIndexesToMessages: migrated (0.0050s) =============

== 20091017212938 AddMissingIndexesToRepositories: migrating ==================
-- add_index(:repositories, :project_id)
   -> 0.0027s
== 20091017212938 AddMissingIndexesToRepositories: migrated (0.0027s) =========

== 20091017213027 AddMissingIndexesToComments: migrating ======================
-- add_index(:comments, [:commented_id, :commented_type])
   -> 0.0024s
-- add_index(:comments, :author_id)
   -> 0.0024s
== 20091017213027 AddMissingIndexesToComments: migrated (0.0049s) =============

== 20091017213113 AddMissingIndexesToEnumerations: migrating ==================
-- add_index(:enumerations, [:id, :type])
   -> 0.0026s
== 20091017213113 AddMissingIndexesToEnumerations: migrated (0.0027s) =========

== 20091017213151 AddMissingIndexesToWikiPages: migrating =====================
-- add_index(:wiki_pages, :wiki_id)
   -> 0.0027s
-- add_index(:wiki_pages, :parent_id)
   -> 0.0026s
== 20091017213151 AddMissingIndexesToWikiPages: migrated (0.0054s) ============

== 20091017213228 AddMissingIndexesToWatchers: migrating ======================
-- add_index(:watchers, :user_id)
   -> 0.0025s
-- add_index(:watchers, [:watchable_id, :watchable_type])
   -> 0.0023s
== 20091017213228 AddMissingIndexesToWatchers: migrated (0.0050s) =============

== 20091017213257 AddMissingIndexesToAuthSources: migrating ===================
-- add_index(:auth_sources, [:id, :type])
   -> 0.0025s
== 20091017213257 AddMissingIndexesToAuthSources: migrated (0.0026s) ==========

== 20091017213332 AddMissingIndexesToDocuments: migrating =====================
-- add_index(:documents, :category_id)
   -> 0.0025s
== 20091017213332 AddMissingIndexesToDocuments: migrated (0.0026s) ============

== 20091017213444 AddMissingIndexesToTokens: migrating ========================
-- add_index(:tokens, :user_id)
   -> 0.0025s
== 20091017213444 AddMissingIndexesToTokens: migrated (0.0025s) ===============

== 20091017213536 AddMissingIndexesToChangesets: migrating ====================
-- add_index(:changesets, :user_id)
   -> 0.0025s
-- add_index(:changesets, :repository_id)
   -> 0.0025s
== 20091017213536 AddMissingIndexesToChangesets: migrated (0.0051s) ===========

== 20091017213642 AddMissingIndexesToIssueCategories: migrating ===============
-- add_index(:issue_categories, :assigned_to_id)
   -> 0.0025s
== 20091017213642 AddMissingIndexesToIssueCategories: migrated (0.0026s) ======

== 20091017213716 AddMissingIndexesToMemberRoles: migrating ===================
-- add_index(:member_roles, :member_id)
   -> 0.0026s
-- add_index(:member_roles, :role_id)
   -> 0.0023s
== 20091017213716 AddMissingIndexesToMemberRoles: migrated (0.0050s) ==========

== 20091017213757 AddMissingIndexesToBoards: migrating ========================
-- add_index(:boards, :last_message_id)
   -> 0.0025s
== 20091017213757 AddMissingIndexesToBoards: migrated (0.0026s) ===============

== 20091017213835 AddMissingIndexesToUserPreferences: migrating ===============
-- add_index(:user_preferences, :user_id)
   -> 0.0025s
== 20091017213835 AddMissingIndexesToUserPreferences: migrated (0.0026s) ======

== 20091017213910 AddMissingIndexesToIssues: migrating ========================
-- add_index(:issues, :status_id)
   -> 0.0025s
-- add_index(:issues, :category_id)
   -> 0.0024s
-- add_index(:issues, :assigned_to_id)
   -> 0.0025s
-- add_index(:issues, :fixed_version_id)
   -> 0.0025s
-- add_index(:issues, :tracker_id)
   -> 0.0025s
-- add_index(:issues, :priority_id)
   -> 0.0024s
-- add_index(:issues, :author_id)
   -> 0.0024s
== 20091017213910 AddMissingIndexesToIssues: migrated (0.0175s) ===============

== 20091017214015 AddMissingIndexesToMembers: migrating =======================
-- add_index(:members, :user_id)
   -> 0.0028s
-- add_index(:members, :project_id)
   -> 0.0024s
== 20091017214015 AddMissingIndexesToMembers: migrated (0.0054s) ==============

== 20091017214107 AddMissingIndexesToCustomFields: migrating ==================
-- add_index(:custom_fields, [:id, :type])
   -> 0.0026s
== 20091017214107 AddMissingIndexesToCustomFields: migrated (0.0026s) =========

== 20091017214136 AddMissingIndexesToQueries: migrating =======================
-- add_index(:queries, :project_id)
   -> 0.0026s
-- add_index(:queries, :user_id)
   -> 0.0025s
== 20091017214136 AddMissingIndexesToQueries: migrated (0.0052s) ==============

== 20091017214236 AddMissingIndexesToTimeEntries: migrating ===================
-- add_index(:time_entries, :activity_id)
   -> 0.0026s
-- add_index(:time_entries, :user_id)
   -> 0.0024s
== 20091017214236 AddMissingIndexesToTimeEntries: migrated (0.0051s) ==========

== 20091017214308 AddMissingIndexesToNews: migrating ==========================
-- add_index(:news, :author_id)
   -> 0.0025s
== 20091017214308 AddMissingIndexesToNews: migrated (0.0026s) =================

== 20091017214336 AddMissingIndexesToUsers: migrating =========================
-- add_index(:users, [:id, :type])
   -> 0.0026s
-- add_index(:users, :auth_source_id)
   -> 0.0024s
== 20091017214336 AddMissingIndexesToUsers: migrated (0.0051s) ================

== 20091017214406 AddMissingIndexesToAttachments: migrating ===================
-- add_index(:attachments, [:container_id, :container_type])
   -> 0.0025s
-- add_index(:attachments, :author_id)
   -> 0.0024s
== 20091017214406 AddMissingIndexesToAttachments: migrated (0.0050s) ==========

== 20091017214440 AddMissingIndexesToWikiContents: migrating ==================
-- add_index(:wiki_contents, :author_id)
   -> 0.0025s
== 20091017214440 AddMissingIndexesToWikiContents: migrated (0.0026s) =========

== 20091017214519 AddMissingIndexesToCustomValues: migrating ==================
-- add_index(:custom_values, :custom_field_id)
   -> 0.0025s
== 20091017214519 AddMissingIndexesToCustomValues: migrated (0.0025s) =========

== 20091017214611 AddMissingIndexesToJournals: migrating ======================
-- add_index(:journals, :user_id)
   -> 0.0027s
-- add_index(:journals, :journalized_id)
   -> 0.0024s
== 20091017214611 AddMissingIndexesToJournals: migrated (0.0052s) =============

== 20091017214644 AddMissingIndexesToIssueRelations: migrating ================
-- add_index(:issue_relations, :issue_from_id)
   -> 0.0025s
-- add_index(:issue_relations, :issue_to_id)
   -> 0.0026s
== 20091017214644 AddMissingIndexesToIssueRelations: migrated (0.0053s) =======

== 20091017214720 AddMissingIndexesToWikiRedirects: migrating =================
-- add_index(:wiki_redirects, :wiki_id)
   -> 0.0024s
== 20091017214720 AddMissingIndexesToWikiRedirects: migrated (0.0024s) ========

== 20091017214750 AddMissingIndexesToCustomFieldsTrackers: migrating ==========
-- add_index(:custom_fields_trackers, [:custom_field_id, :tracker_id])
   -> 0.0026s
== 20091017214750 AddMissingIndexesToCustomFieldsTrackers: migrated (0.0027s) =

== 20091025163651 AddActivityIndexes: migrating ===============================
-- add_index(:journals, :created_on)
   -> 0.0026s
-- add_index(:changesets, :committed_on)
   -> 0.0024s
-- add_index(:wiki_content_versions, :updated_on)
   -> 0.0024s
-- add_index(:messages, :created_on)
   -> 0.0024s
-- add_index(:issues, :created_on)
   -> 0.0024s
-- add_index(:news, :created_on)
   -> 0.0026s
-- add_index(:attachments, :created_on)
   -> 0.0024s
-- add_index(:documents, :created_on)
   -> 0.0024s
-- add_index(:time_entries, :created_on)
   -> 0.0023s
== 20091025163651 AddActivityIndexes: migrated (0.0223s) ======================

== 20091108092559 AddVersionsStatus: migrating ================================
-- add_column(:versions, :status, :string, {:default=>"open"})
   -> 0.0011s
== 20091108092559 AddVersionsStatus: migrated (0.0097s) =======================

== 20091114105931 AddViewIssuesPermission: migrating ==========================
== 20091114105931 AddViewIssuesPermission: migrated (0.0121s) =================

== 20091123212029 AddDefaultDoneRatioToIssueStatus: migrating =================
-- add_column(:issue_statuses, :default_done_ratio, :integer, {})
   -> 0.0006s
== 20091123212029 AddDefaultDoneRatioToIssueStatus: migrated (0.0007s) ========

== 20091205124427 AddVersionsSharing: migrating ===============================
-- add_column(:versions, :sharing, :string, {:default=>"none", :null=>false})
   -> 0.0009s
-- add_index(:versions, :sharing)
   -> 0.0033s
== 20091205124427 AddVersionsSharing: migrated (0.0044s) ======================

== 20091220183509 AddLftAndRgtIndexesToProjects: migrating ====================
-- add_index(:projects, :lft)
   -> 0.0035s
-- add_index(:projects, :rgt)
   -> 0.0024s
== 20091220183509 AddLftAndRgtIndexesToProjects: migrated (0.0060s) ===========

== 20091220183727 AddIndexToSettingsName: migrating ===========================
-- add_index(:settings, :name)
   -> 0.0027s
== 20091220183727 AddIndexToSettingsName: migrated (0.0027s) ==================

== 20091220184736 AddIndexesToIssueStatus: migrating ==========================
-- add_index(:issue_statuses, :position)
   -> 0.0026s
-- add_index(:issue_statuses, :is_closed)
   -> 0.0023s
-- add_index(:issue_statuses, :is_default)
   -> 0.0024s
== 20091220184736 AddIndexesToIssueStatus: migrated (0.0074s) =================

== 20091225164732 RemoveEnumerationsOpt: migrating ============================
-- remove_column(:enumerations, :opt)
   -> 0.0006s
== 20091225164732 RemoveEnumerationsOpt: migrated (0.0007s) ===================

== 20091227112908 ChangeWikiContentsTextLimit: migrating ======================
== 20091227112908 ChangeWikiContentsTextLimit: migrated (0.0000s) =============

== 20100129193402 ChangeUsersMailNotificationToString: migrating ==============
-- rename_column(:users, :mail_notification, :mail_notification_bool)
   -> 0.0022s
-- add_column(:users, :mail_notification, :string, {:default=>"", :null=>false})
   -> 0.0009s
-- remove_column(:users, :mail_notification_bool)
   -> 0.0006s
== 20100129193402 ChangeUsersMailNotificationToString: migrated (0.0065s) =====

== 20100129193813 UpdateMailNotificationValues: migrating =====================
== 20100129193813 UpdateMailNotificationValues: migrated (0.0000s) ============

== 20100221100219 AddIndexOnChangesetsScmid: migrating ========================
-- add_index(:changesets, [:repository_id, :scmid], {:name=>:changesets_repos_scmid})
   -> 0.0034s
== 20100221100219 AddIndexOnChangesetsScmid: migrated (0.0035s) ===============

== 20100313132032 AddIssuesNestedSetsColumns: migrating =======================
-- add_column(:issues, :parent_id, :integer, {:default=>nil})
   -> 0.0008s
-- add_column(:issues, :root_id, :integer, {:default=>nil})
   -> 0.0018s
-- add_column(:issues, :lft, :integer, {:default=>nil})
   -> 0.0008s
-- add_column(:issues, :rgt, :integer, {:default=>nil})
   -> 0.0007s
== 20100313132032 AddIssuesNestedSetsColumns: migrated (0.0500s) ==============

== 20100313171051 AddIndexOnIssuesNestedSet: migrating ========================
-- add_index(:issues, [:root_id, :lft, :rgt])
   -> 0.0026s
== 20100313171051 AddIndexOnIssuesNestedSet: migrated (0.0027s) ===============

== 20100705164950 ChangeChangesPathLengthLimit: migrating =====================
-- change_column(:changes, :path, :text, {})
   -> 0.0008s
-- change_column(:changes, :path, :text, {})
   -> 0.0005s
-- change_column(:changes, :from_path, :text, {})
   -> 0.0004s
== 20100705164950 ChangeChangesPathLengthLimit: migrated (0.0052s) ============

== 20100819172912 EnableCalendarAndGanttModulesWhereAppropriate: migrating ====
== 20100819172912 EnableCalendarAndGanttModulesWhereAppropriate: migrated (0.0080s) 

== 20101104182107 AddUniqueIndexOnMembers: migrating ==========================
-- add_index(:members, [:user_id, :project_id], {:unique=>true})
   -> 0.0025s
== 20101104182107 AddUniqueIndexOnMembers: migrated (0.0065s) =================

== 20101107130441 AddCustomFieldsVisible: migrating ===========================
-- add_column(:custom_fields, :visible, :boolean, {:null=>false, :default=>true})
   -> 0.0011s
== 20101107130441 AddCustomFieldsVisible: migrated (0.0026s) ==================

== 20101114115114 ChangeProjectsNameLimit: migrating ==========================
-- change_column(:projects, :name, :string, {:limit=>nil})
   -> 0.0008s
== 20101114115114 ChangeProjectsNameLimit: migrated (0.0057s) =================

== 20101114115359 ChangeProjectsIdentifierLimit: migrating ====================
-- change_column(:projects, :identifier, :string, {:limit=>nil})
   -> 0.0007s
== 20101114115359 ChangeProjectsIdentifierLimit: migrated (0.0008s) ===========

== 20110220160626 AddWorkflowsAssigneeAndAuthor: migrating ====================
-- add_column(:workflows, :assignee, :boolean, {:null=>false, :default=>false})
   -> 0.0009s
-- add_column(:workflows, :author, :boolean, {:null=>false, :default=>false})
   -> 0.0008s
== 20110220160626 AddWorkflowsAssigneeAndAuthor: migrated (0.0113s) ===========

== 20110223180944 AddUsersSalt: migrating =====================================
-- add_column(:users, :salt, :string, {:limit=>64})
   -> 0.0012s
== 20110223180944 AddUsersSalt: migrated (0.0013s) ============================

== 20110223180953 SaltUserPasswords: migrating ================================
-- Salting user passwords, this may take some time...
   -> 0.0498s
== 20110223180953 SaltUserPasswords: migrated (0.0499s) =======================

== 20110224000000 AddRepositoriesPathEncoding: migrating ======================
-- add_column(:repositories, :path_encoding, :string, {:limit=>64, :default=>nil})
   -> 0.0011s
== 20110224000000 AddRepositoriesPathEncoding: migrated (0.0012s) =============

== 20110226120112 ChangeRepositoriesPasswordLimit: migrating ==================
-- change_column(:repositories, :password, :string, {:limit=>nil})
   -> 0.0007s
== 20110226120112 ChangeRepositoriesPasswordLimit: migrated (0.0026s) =========

== 20110226120132 ChangeAuthSourcesAccountPasswordLimit: migrating ============
-- change_column(:auth_sources, :account_password, :string, {:limit=>nil})
   -> 0.0007s
== 20110226120132 ChangeAuthSourcesAccountPasswordLimit: migrated (0.0027s) ===

== 20110227125750 ChangeJournalDetailsValuesToText: migrating =================
-- change_column(:journal_details, :old_value, :text, {})
   -> 0.0006s
-- change_column(:journal_details, :value, :text, {})
   -> 0.0005s
== 20110227125750 ChangeJournalDetailsValuesToText: migrated (0.0012s) ========

== 20110228000000 AddRepositoriesLogEncoding: migrating =======================
-- add_column(:repositories, :log_encoding, :string, {:limit=>64, :default=>nil})
   -> 0.0009s
== 20110228000000 AddRepositoriesLogEncoding: migrated (0.0010s) ==============

== 20110228000100 CopyRepositoriesLogEncoding: migrating ======================
== 20110228000100 CopyRepositoriesLogEncoding: migrated (0.0067s) =============

== 20110401192910 AddIndexToUsersType: migrating ==============================
-- add_index(:users, :type)
   -> 0.0028s
== 20110401192910 AddIndexToUsersType: migrated (0.0029s) =====================

== 20110408103312 AddRolesIssuesVisibility: migrating =========================
-- add_column(:roles, :issues_visibility, :string, {:limit=>30, :default=>"default", :null=>false})
   -> 0.0011s
== 20110408103312 AddRolesIssuesVisibility: migrated (0.0011s) ================

== 20110412065600 AddIssuesIsPrivate: migrating ===============================
-- add_column(:issues, :is_private, :boolean, {:default=>false, :null=>false})
   -> 0.0009s
== 20110412065600 AddIssuesIsPrivate: migrated (0.0010s) ======================

== 20110511000000 AddRepositoriesExtraInfo: migrating =========================
-- add_column(:repositories, :extra_info, :text, {})
   -> 0.0006s
== 20110511000000 AddRepositoriesExtraInfo: migrated (0.0007s) ================

== 20110902000000 CreateChangesetParents: migrating ===========================
-- create_table(:changeset_parents, {:id=>false})
   -> 0.0009s
-- add_index(:changeset_parents, [:changeset_id], {:unique=>false, :name=>:changeset_parents_changeset_ids})
   -> 0.0025s
-- add_index(:changeset_parents, [:parent_id], {:unique=>false, :name=>:changeset_parents_parent_ids})
   -> 0.0023s
== 20110902000000 CreateChangesetParents: migrated (0.0059s) ==================

== 20111201201315 AddUniqueIndexToIssueRelations: migrating ===================
-- add_index(:issue_relations, [:issue_from_id, :issue_to_id], {:unique=>true})
   -> 0.0024s
== 20111201201315 AddUniqueIndexToIssueRelations: migrated (0.0034s) ==========

== 20120115143024 AddRepositoriesIdentifier: migrating ========================
-- add_column(:repositories, :identifier, :string, {})
   -> 0.0006s
== 20120115143024 AddRepositoriesIdentifier: migrated (0.0007s) ===============

== 20120115143100 AddRepositoriesIsDefault: migrating =========================
-- add_column(:repositories, :is_default, :boolean, {:default=>false})
   -> 0.0009s
== 20120115143100 AddRepositoriesIsDefault: migrated (0.0010s) ================

== 20120115143126 SetDefaultRepositories: migrating ===========================
== 20120115143126 SetDefaultRepositories: migrated (0.0012s) ==================

== 20120127174243 AddCustomFieldsMultiple: migrating ==========================
-- add_column(:custom_fields, :multiple, :boolean, {:default=>false})
   -> 0.0010s
== 20120127174243 AddCustomFieldsMultiple: migrated (0.0011s) =================

== 20120205111326 ChangeUsersLoginLimit: migrating ============================
-- change_column(:users, :login, :string, {:limit=>nil})
   -> 0.0007s
== 20120205111326 ChangeUsersLoginLimit: migrated (0.0046s) ===================

== 20120223110929 ChangeAttachmentsContainerDefaults: migrating ===============
-- remove_index(:attachments, {:column=>[:container_id, :container_type], :name=>"index_attachments_on_container_id_and_container_type"})
   -> 0.0024s
-- change_column(:attachments, :container_id, :integer, {})
   -> 0.0006s
-- change_column(:attachments, :container_type, :string, {:limit=>30})
   -> 0.0006s
-- add_index(:attachments, [:container_id, :container_type])
   -> 0.0025s
== 20120223110929 ChangeAttachmentsContainerDefaults: migrated (0.0236s) ======

== 20120301153455 AddAuthSourcesFilter: migrating =============================
-- add_column(:auth_sources, :filter, :string, {})
   -> 0.0007s
== 20120301153455 AddAuthSourcesFilter: migrated (0.0007s) ====================

== 20120422150750 ChangeRepositoriesToFullSti: migrating ======================
== 20120422150750 ChangeRepositoriesToFullSti: migrated (0.0006s) =============

== 20120705074331 AddTrackersFieldsBits: migrating ============================
-- add_column(:trackers, :fields_bits, :integer, {:default=>0})
   -> 0.0010s
== 20120705074331 AddTrackersFieldsBits: migrated (0.0011s) ===================

== 20120707064544 AddAuthSourcesTimeout: migrating ============================
-- add_column(:auth_sources, :timeout, :integer, {})
   -> 0.0005s
== 20120707064544 AddAuthSourcesTimeout: migrated (0.0006s) ===================

== 20120714122000 AddWorkflowsType: migrating =================================
-- add_column(:workflows, :type, :string, {:limit=>30})
   -> 0.0006s
== 20120714122000 AddWorkflowsType: migrated (0.0007s) ========================

== 20120714122100 UpdateWorkflowsToSti: migrating =============================
== 20120714122100 UpdateWorkflowsToSti: migrated (0.0007s) ====================

== 20120714122200 AddWorkflowsRuleFields: migrating ===========================
-- add_column(:workflows, :field_name, :string, {:limit=>30})
   -> 0.0006s
-- add_column(:workflows, :rule, :string, {:limit=>30})
   -> 0.0005s
== 20120714122200 AddWorkflowsRuleFields: migrated (0.0012s) ==================

== 20120731164049 AddBoardsParentId: migrating ================================
-- add_column(:boards, :parent_id, :integer, {})
   -> 0.0006s
== 20120731164049 AddBoardsParentId: migrated (0.0007s) =======================

== 20120930112914 AddJournalsPrivateNotes: migrating ==========================
-- add_column(:journals, :private_notes, :boolean, {:default=>false, :null=>false})
   -> 0.0009s
== 20120930112914 AddJournalsPrivateNotes: migrated (0.0010s) =================

== 20121026002032 AddEnumerationsPositionName: migrating ======================
-- add_column(:enumerations, :position_name, :string, {:limit=>30})
   -> 0.0006s
== 20121026002032 AddEnumerationsPositionName: migrated (0.0007s) =============

== 20121026003537 PopulateEnumerationsPositionName: migrating =================
== 20121026003537 PopulateEnumerationsPositionName: migrated (0.0016s) ========

== 20121209123234 AddQueriesType: migrating ===================================
-- add_column(:queries, :type, :string, {})
   -> 0.0006s
== 20121209123234 AddQueriesType: migrated (0.0007s) ==========================

== 20121209123358 UpdateQueriesToSti: migrating ===============================
== 20121209123358 UpdateQueriesToSti: migrated (0.0149s) ======================

== 20121213084931 AddAttachmentsDiskDirectory: migrating ======================
-- add_column(:attachments, :disk_directory, :string, {})
   -> 0.0007s
== 20121213084931 AddAttachmentsDiskDirectory: migrated (0.0007s) =============

== 20130110122628 SplitDocumentsPermissions: migrating ========================
== 20130110122628 SplitDocumentsPermissions: migrated (0.0010s) ===============

== 20130201184705 AddUniqueIndexOnTokensValue: migrating ======================
-- Adding unique index on tokens, this may take some time...
-- add_index(:tokens, :value, {:unique=>true, :name=>"tokens_value"})
   -> 0.0025s
   -> 0.0072s
== 20130201184705 AddUniqueIndexOnTokensValue: migrated (0.0072s) =============

== 20130202090625 AddProjectsInheritMembers: migrating ========================
-- add_column(:projects, :inherit_members, :boolean, {:default=>false, :null=>false})
   -> 0.0010s
== 20130202090625 AddProjectsInheritMembers: migrated (0.0010s) ===============

== 20130207175206 AddUniqueIndexOnCustomFieldsTrackers: migrating =============
-- index_exists?(:custom_fields_trackers, [:custom_field_id, :tracker_id], {:name=>"index_custom_fields_trackers_on_custom_field_id_and_tracker_id"})
   -> 0.0015s
-- remove_index(:custom_fields_trackers, {:column=>[:custom_field_id, :tracker_id], :name=>"index_custom_fields_trackers_on_custom_field_id_and_tracker_id"})
   -> 0.0018s
-- add_index(:custom_fields_trackers, [:custom_field_id, :tracker_id], {:unique=>true})
   -> 0.0025s
== 20130207175206 AddUniqueIndexOnCustomFieldsTrackers: migrated (0.0077s) ====

== 20130207181455 AddUniqueIndexOnCustomFieldsProjects: migrating =============
-- index_exists?(:custom_fields_projects, [:custom_field_id, :project_id], {:name=>"index_custom_fields_projects_on_custom_field_id_and_project_id"})
   -> 0.0014s
-- remove_index(:custom_fields_projects, {:column=>[:custom_field_id, :project_id], :name=>"index_custom_fields_projects_on_custom_field_id_and_project_id"})
   -> 0.0017s
-- add_index(:custom_fields_projects, [:custom_field_id, :project_id], {:unique=>true})
   -> 0.0024s
== 20130207181455 AddUniqueIndexOnCustomFieldsProjects: migrated (0.0071s) ====

== 20130215073721 ChangeUsersLastnameLengthTo255: migrating ===================
-- change_column(:users, :lastname, :string, {:limit=>255})
   -> 0.0009s
== 20130215073721 ChangeUsersLastnameLengthTo255: migrated (0.0047s) ==========

== 20130215111127 AddIssuesClosedOn: migrating ================================
-- add_column(:issues, :closed_on, :datetime, {:default=>nil})
   -> 0.0009s
== 20130215111127 AddIssuesClosedOn: migrated (0.0010s) =======================

== 20130215111141 PopulateIssuesClosedOn: migrating ===========================
== 20130215111141 PopulateIssuesClosedOn: migrated (0.0011s) ==================

== 20130217094251 RemoveIssuesDefaultFkValues: migrating ======================
-- change_column_default(:issues, :tracker_id, nil)
   -> 0.0022s
-- change_column_default(:issues, :project_id, nil)
   -> 0.0019s
-- change_column_default(:issues, :status_id, nil)
   -> 0.0020s
-- change_column_default(:issues, :assigned_to_id, nil)
   -> 0.0019s
-- change_column_default(:issues, :priority_id, nil)
   -> 0.0020s
-- change_column_default(:issues, :author_id, nil)
   -> 0.0019s
== 20130217094251 RemoveIssuesDefaultFkValues: migrated (0.0122s) =============

== 20130602092539 CreateQueriesRoles: migrating ===============================
-- create_table(:queries_roles, {:id=>false})
   -> 0.0009s
-- add_index(:queries_roles, [:query_id, :role_id], {:unique=>true, :name=>:queries_roles_ids})
   -> 0.0026s
== 20130602092539 CreateQueriesRoles: migrated (0.0037s) ======================

== 20130710182539 AddQueriesVisibility: migrating =============================
-- add_column(:queries, :visibility, :integer, {:default=>0})
   -> 0.0010s
-- remove_column(:queries, :is_public)
   -> 0.0005s
== 20130710182539 AddQueriesVisibility: migrated (0.0025s) ====================

== 20130713104233 CreateCustomFieldsRoles: migrating ==========================
-- create_table(:custom_fields_roles, {:id=>false})
   -> 0.0009s
-- add_index(:custom_fields_roles, [:custom_field_id, :role_id], {:unique=>true, :name=>:custom_fields_roles_ids})
   -> 0.0024s
== 20130713104233 CreateCustomFieldsRoles: migrated (0.0044s) =================

== 20130713111657 AddQueriesOptions: migrating ================================
-- add_column(:queries, :options, :text, {})
   -> 0.0007s
== 20130713111657 AddQueriesOptions: migrated (0.0007s) =======================

== 20130729070143 AddUsersMustChangePasswd: migrating =========================
-- add_column(:users, :must_change_passwd, :boolean, {:default=>false, :null=>false})
   -> 0.0010s
== 20130729070143 AddUsersMustChangePasswd: migrated (0.0010s) ================

== 20130911193200 RemoveEolsFromAttachmentsFilename: migrating ================
== 20130911193200 RemoveEolsFromAttachmentsFilename: migrated (0.0028s) =======

== 20131004113137 SupportForMultipleCommitKeywords: migrating =================
== 20131004113137 SupportForMultipleCommitKeywords: migrated (0.0029s) ========

== 20131005100610 AddRepositoriesCreatedOn: migrating =========================
-- add_column(:repositories, :created_on, :timestamp, {})
   -> 0.0007s
== 20131005100610 AddRepositoriesCreatedOn: migrated (0.0008s) ================

== 20131124175346 AddCustomFieldsFormatStore: migrating =======================
-- add_column(:custom_fields, :format_store, :text, {})
   -> 0.0006s
== 20131124175346 AddCustomFieldsFormatStore: migrated (0.0007s) ==============

== 20131210180802 AddCustomFieldsDescription: migrating =======================
-- add_column(:custom_fields, :description, :text, {})
   -> 0.0006s
== 20131210180802 AddCustomFieldsDescription: migrated (0.0007s) ==============

== 20131214094309 RemoveCustomFieldsMinMaxLengthDefaultValues: migrating ======
-- change_column(:custom_fields, :min_length, :int, {})
   -> 0.0008s
-- change_column(:custom_fields, :max_length, :int, {})
   -> 0.0010s
== 20131214094309 RemoveCustomFieldsMinMaxLengthDefaultValues: migrated (0.0078s) 

== 20131215104612 StoreRelationTypeInJournalDetails: migrating ================
== 20131215104612 StoreRelationTypeInJournalDetails: migrated (0.0090s) =======

== 20131218183023 DeleteOrphanTimeEntriesCustomValues: migrating ==============
== 20131218183023 DeleteOrphanTimeEntriesCustomValues: migrated (0.0012s) =====

== 20140228130325 ChangeChangesetsCommentsLimit: migrating ====================
== 20140228130325 ChangeChangesetsCommentsLimit: migrated (0.0000s) ===========

== 20140903143914 AddPasswordChangedAtToUser: migrating =======================
-- add_column(:users, :passwd_changed_on, :datetime, {})
   -> 0.0007s
== 20140903143914 AddPasswordChangedAtToUser: migrated (0.0007s) ==============

== 20140920094058 InsertBuiltinGroups: migrating ==============================
== 20140920094058 InsertBuiltinGroups: migrated (0.0726s) =====================

== 20141029181752 AddTrackersDefaultStatusId: migrating =======================
-- add_column(:trackers, :default_status_id, :integer, {})
   -> 0.0007s
== 20141029181752 AddTrackersDefaultStatusId: migrated (0.0022s) ==============

== 20141029181824 RemoveIssueStatusesIsDefault: migrating =====================
-- remove_column(:issue_statuses, :is_default)
   -> 0.0006s
== 20141029181824 RemoveIssueStatusesIsDefault: migrated (0.0007s) ============

== 20141109112308 AddRolesUsersVisibility: migrating ==========================
-- add_column(:roles, :users_visibility, :string, {:limit=>30, :default=>"all", :null=>false})
   -> 0.0011s
== 20141109112308 AddRolesUsersVisibility: migrated (0.0012s) =================

== 20141122124142 AddWikiRedirectsRedirectsToWikiId: migrating ================
-- add_column(:wiki_redirects, :redirects_to_wiki_id, :integer, {})
   -> 0.0006s
-- change_column(:wiki_redirects, :redirects_to_wiki_id, :integer, {})
   -> 0.0006s
== 20141122124142 AddWikiRedirectsRedirectsToWikiId: migrated (0.0076s) =======

== 20150113194759 CreateEmailAddresses: migrating =============================
-- create_table(:email_addresses, {:id=>:integer})
   -> 0.0044s
== 20150113194759 CreateEmailAddresses: migrated (0.0045s) ====================

== 20150113211532 PopulateEmailAddresses: migrating ===========================
== 20150113211532 PopulateEmailAddresses: migrated (0.0031s) ==================

== 20150113213922 RemoveUsersMail: migrating ==================================
-- remove_column(:users, :mail)
   -> 0.0007s
== 20150113213922 RemoveUsersMail: migrated (0.0007s) =========================

== 20150113213955 AddEmailAddressesUserIdIndex: migrating =====================
-- add_index(:email_addresses, :user_id)
   -> 0.0025s
== 20150113213955 AddEmailAddressesUserIdIndex: migrated (0.0026s) ============

== 20150208105930 ReplaceMoveIssuesPermission: migrating ======================
== 20150208105930 ReplaceMoveIssuesPermission: migrated (0.0010s) =============

== 20150510083747 ChangeDocumentsTitleLimit: migrating ========================
-- change_column(:documents, :title, :string, {:limit=>nil})
   -> 0.0007s
== 20150510083747 ChangeDocumentsTitleLimit: migrated (0.0041s) ===============

== 20150525103953 ClearEstimatedHoursOnParentIssues: migrating ================
== 20150525103953 ClearEstimatedHoursOnParentIssues: migrated (0.0034s) =======

== 20150526183158 AddRolesTimeEntriesVisibility: migrating ====================
-- add_column(:roles, :time_entries_visibility, :string, {:limit=>30, :default=>"all", :null=>false})
   -> 0.0011s
== 20150526183158 AddRolesTimeEntriesVisibility: migrated (0.0012s) ===========

== 20150528084820 AddRolesAllRolesManaged: migrating ==========================
-- add_column(:roles, :all_roles_managed, :boolean, {:default=>true, :null=>false})
   -> 0.0010s
== 20150528084820 AddRolesAllRolesManaged: migrated (0.0011s) =================

== 20150528092912 CreateRolesManagedRoles: migrating ==========================
-- create_table(:roles_managed_roles, {:id=>false})
   -> 0.0010s
== 20150528092912 CreateRolesManagedRoles: migrated (0.0010s) =================

== 20150528093249 AddUniqueIndexOnRolesManagedRoles: migrating ================
-- add_index(:roles_managed_roles, [:role_id, :managed_role_id], {:unique=>true})
   -> 0.0028s
== 20150528093249 AddUniqueIndexOnRolesManagedRoles: migrated (0.0029s) =======

== 20150725112753 InsertAllowedStatusesForNewIssues: migrating ================
== 20150725112753 InsertAllowedStatusesForNewIssues: migrated (0.0026s) =======

== 20150730122707 CreateImports: migrating ====================================
-- create_table(:imports, {:id=>:integer})
   -> 0.0040s
== 20150730122707 CreateImports: migrated (0.0041s) ===========================

== 20150730122735 CreateImportItems: migrating ================================
-- create_table(:import_items, {:id=>:integer})
   -> 0.0035s
== 20150730122735 CreateImportItems: migrated (0.0036s) =======================

== 20150921204850 ChangeTimeEntriesCommentsLimitTo1024: migrating =============
-- change_column(:time_entries, :comments, :string, {:limit=>1024})
   -> 0.0019s
== 20150921204850 ChangeTimeEntriesCommentsLimitTo1024: migrated (0.0020s) ====

== 20150921210243 ChangeWikiContentsCommentsLimitTo1024: migrating ============
-- change_column(:wiki_content_versions, :comments, :string, {:limit=>1024})
   -> 0.0007s
-- change_column(:wiki_contents, :comments, :string, {:limit=>1024})
   -> 0.0007s
== 20150921210243 ChangeWikiContentsCommentsLimitTo1024: migrated (0.0050s) ===

== 20151020182334 ChangeAttachmentsFilesizeLimitTo8: migrating ================
-- change_column(:attachments, :filesize, :integer, {:limit=>8})
   -> 0.0060s
== 20151020182334 ChangeAttachmentsFilesizeLimitTo8: migrated (0.0099s) =======

== 20151020182731 FixCommaInUserFormatSettingValue: migrating =================
== 20151020182731 FixCommaInUserFormatSettingValue: migrated (0.0009s) ========

== 20151021184614 ChangeIssueCategoriesNameLimitTo60: migrating ===============
-- change_column(:issue_categories, :name, :string, {:limit=>60})
   -> 0.0008s
== 20151021184614 ChangeIssueCategoriesNameLimitTo60: migrated (0.0042s) ======

== 20151021185456 ChangeAuthSourcesFilterToText: migrating ====================
-- change_column(:auth_sources, :filter, :text, {})
   -> 0.0006s
== 20151021185456 ChangeAuthSourcesFilterToText: migrated (0.0007s) ===========

== 20151021190616 ChangeUserPreferencesHideMailDefaultToTrue: migrating =======
-- change_column(:user_preferences, :hide_mail, :boolean, {})
   -> 0.0010s
== 20151021190616 ChangeUserPreferencesHideMailDefaultToTrue: migrated (0.0027s) 

== 20151024082034 AddTokensUpdatedOn: migrating ===============================
-- add_column(:tokens, :updated_on, :timestamp, {})
   -> 0.0006s
== 20151024082034 AddTokensUpdatedOn: migrated (0.0030s) ======================

== 20151025072118 CreateCustomFieldEnumerations: migrating ====================
-- create_table(:custom_field_enumerations, {:id=>:integer})
   -> 0.0038s
== 20151025072118 CreateCustomFieldEnumerations: migrated (0.0039s) ===========

== 20151031095005 AddProjectsDefaultVersionId: migrating ======================
-- column_exists?(:projects, :default_version_id, :integer)
   -> 0.0017s
-- add_column(:projects, :default_version_id, :integer, {:default=>nil})
   -> 0.0008s
== 20151031095005 AddProjectsDefaultVersionId: migrated (0.0026s) =============

== 20160404080304 ForcePasswordResetDuringSetup: migrating ====================
== 20160404080304 ForcePasswordResetDuringSetup: migrated (0.0011s) ===========

== 20160416072926 RemovePositionDefaults: migrating ===========================
-- change_column("boards", :position, :integer, {})
   -> 0.0007s
-- change_column("custom_fields", :position, :integer, {})
   -> 0.0007s
-- change_column("enumerations", :position, :integer, {})
   -> 0.0006s
-- change_column("issue_statuses", :position, :integer, {})
   -> 0.0010s
-- change_column("roles", :position, :integer, {})
   -> 0.0007s
-- change_column("trackers", :position, :integer, {})
   -> 0.0009s
== 20160416072926 RemovePositionDefaults: migrated (0.0153s) ==================

== 20160529063352 AddRolesSettings: migrating =================================
-- add_column(:roles, :settings, :text, {})
   -> 0.0007s
== 20160529063352 AddRolesSettings: migrated (0.0007s) ========================

== 20161001122012 AddTrackerIdIndexToWorkflows: migrating =====================
-- add_index(:workflows, :tracker_id)
   -> 0.0024s
== 20161001122012 AddTrackerIdIndexToWorkflows: migrated (0.0025s) ============

== 20161002133421 AddIndexOnMemberRolesInheritedFrom: migrating ===============
-- add_index(:member_roles, :inherited_from)
   -> 0.0024s
== 20161002133421 AddIndexOnMemberRolesInheritedFrom: migrated (0.0024s) ======

== 20161010081301 ChangeIssuesDescriptionLimit: migrating =====================
== 20161010081301 ChangeIssuesDescriptionLimit: migrated (0.0000s) ============

== 20161010081528 ChangeJournalDetailsValueLimit: migrating ===================
== 20161010081528 ChangeJournalDetailsValueLimit: migrated (0.0000s) ==========

== 20161010081600 ChangeJournalsNotesLimit: migrating =========================
== 20161010081600 ChangeJournalsNotesLimit: migrated (0.0000s) ================

== 20161126094932 AddIndexOnChangesetsIssuesIssueId: migrating ================
-- add_index(:changesets_issues, :issue_id)
   -> 0.0024s
== 20161126094932 AddIndexOnChangesetsIssuesIssueId: migrated (0.0024s) =======

== 20161220091118 AddIndexOnIssuesParentId: migrating =========================
-- add_index(:issues, :parent_id)
   -> 0.0023s
== 20161220091118 AddIndexOnIssuesParentId: migrated (0.0024s) ================

== 20170207050700 AddIndexOnDiskFilenameToAttachments: migrating ==============
-- add_index(:attachments, :disk_filename)
   -> 0.0024s
== 20170207050700 AddIndexOnDiskFilenameToAttachments: migrated (0.0025s) =====

== 20170302015225 ChangeAttachmentsDigestLimitTo64: migrating =================
-- change_column(:attachments, :digest, :string, {:limit=>64})
   -> 0.0008s
== 20170302015225 ChangeAttachmentsDigestLimitTo64: migrated (0.0008s) ========

== 20170309214320 AddProjectDefaultAssignedToId: migrating ====================
-- add_column(:projects, :default_assigned_to_id, :integer, {:default=>nil})
   -> 0.0009s
-- column_exists?(:projects, :default_assignee_id, :integer)
   -> 0.0016s
== 20170309214320 AddProjectDefaultAssignedToId: migrated (0.0026s) ===========

== 20170320051650 ChangeRepositoriesExtraInfoLimit: migrating =================
== 20170320051650 ChangeRepositoriesExtraInfoLimit: migrated (0.0000s) ========

== 20170418090031 AddViewNewsToAllExistingRoles: migrating ====================
== 20170418090031 AddViewNewsToAllExistingRoles: migrated (0.0053s) ===========

== 20170419144536 AddViewMessagesToAllExistingRoles: migrating ================
== 20170419144536 AddViewMessagesToAllExistingRoles: migrated (0.0052s) =======

== 20170723112801 RenameCommentsToContent: migrating ==========================
-- rename_column(:comments, :comments, :content)
   -> 0.0021s
== 20170723112801 RenameCommentsToContent: migrated (0.0022s) =================

== 20180913072918 AddVerifyPeerToAuthSources: migrating =======================
-- change_table(:auth_sources)
   -> 0.0010s
== 20180913072918 AddVerifyPeerToAuthSources: migrated (0.0010s) ==============

== 20180923082945 ChangeSqliteBooleansTo0And1: migrating ======================
== 20180923082945 ChangeSqliteBooleansTo0And1: migrated (0.0000s) =============

== 20180923091603 ChangeSqliteBooleansDefault: migrating ======================
== 20180923091603 ChangeSqliteBooleansDefault: migrated (0.0000s) =============

[root@redmine redmine-4.0.4]# echo $?
0
[root@redmine redmine-4.0.4]# 
~~~


>**Note:** 这个过程中如果有数据库连接不通的问题，会有如下报错

~~~
[root@redmine redmine-4.0.4]# RAILS_ENV=production bundle exec rake db:migrate
rake aborted!
PG::ConnectionBad: FATAL:  Ident authentication failed for user "redmine"
/usr/local/rvm/gems/ruby-2.6.3/gems/pg-1.1.4/lib/pg.rb:56:in `initialize'
/usr/local/rvm/gems/ruby-2.6.3/gems/pg-1.1.4/lib/pg.rb:56:in `new'
/usr/local/rvm/gems/ruby-2.6.3/gems/pg-1.1.4/lib/pg.rb:56:in `connect'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/postgresql_adapter.rb:692:in `connect'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/postgresql_adapter.rb:223:in `initialize'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/postgresql_adapter.rb:48:in `new'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/postgresql_adapter.rb:48:in `postgresql_connection'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/abstract/connection_pool.rb:811:in `new_connection'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/abstract/connection_pool.rb:855:in `checkout_new_connection'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/abstract/connection_pool.rb:834:in `try_to_checkout_new_connection'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/abstract/connection_pool.rb:795:in `acquire_connection'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/abstract/connection_pool.rb:523:in `checkout'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/abstract/connection_pool.rb:382:in `connection'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_adapters/abstract/connection_pool.rb:1014:in `retrieve_connection'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_handling.rb:118:in `retrieve_connection'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/connection_handling.rb:90:in `connection'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/tasks/database_tasks.rb:172:in `migrate'
/usr/local/rvm/gems/ruby-2.6.3/gems/activerecord-5.2.3/lib/active_record/railties/databases.rake:60:in `block (2 levels) in <top (required)>'
/usr/local/rvm/gems/ruby-2.6.3/gems/rake-12.3.3/exe/rake:27:in `<top (required)>'
/usr/local/rvm/gems/ruby-2.6.3/bin/ruby_executable_hooks:24:in `eval'
/usr/local/rvm/gems/ruby-2.6.3/bin/ruby_executable_hooks:24:in `<main>'
Tasks: TOP => db:migrate
(See full trace by running task with --trace)
[root@redmine redmine-4.0.4]# 
~~~

## 导入默认数据

这个过程中 **Redmine** 会提示设定语言，也可以在运行命令的前面加上 **REDMINE_LANG** 环境变量来让整个设定过程静默完成

~~~
[root@redmine redmine-4.0.4]# RAILS_ENV=production bundle exec rake redmine:load_default_data

Select language: ar, az, bg, bs, ca, cs, da, de, el, en, en-GB, es, es-PA, et, eu, fa, fi, fr, gl, he, hr, hu, id, it, ja, ko, lt, lv, mk, mn, nl, no, pl, pt, pt-BR, ro, ru, sk, sl, sq, sr, sr-YU, sv, th, tr, uk, vi, zh, zh-TW [en] en
====================================
Default configuration data loaded.
[root@redmine redmine-4.0.4]# echo $?
0
[root@redmine redmine-4.0.4]# 
~~~

>**Tip:** 可以使用这个方式来静默安装

~~~
RAILS_ENV=production REDMINE_LANG=fr bundle exec rake redmine:load_default_data
~~~

查看一下库里的内容

~~~
[root@redmine ~]# psql -U redmine -h 127.0.0.1
Password for user redmine: 
psql (9.2.24, server 11.5)
WARNING: psql version 9.2, server version 11.0.
         Some psql features might not work.
Type "help" for help.

redmine=> \d
                            List of relations
 Schema |                    Name                    |   Type   |  Owner  
--------|--------------------------------------------|----------|---------
 public | ar_internal_metadata                       | table    | redmine
 public | attachments                                | table    | redmine
 public | attachments_id_seq                         | sequence | redmine
 public | auth_sources                               | table    | redmine
 public | auth_sources_id_seq                        | sequence | redmine
 public | boards                                     | table    | redmine
 public | boards_id_seq                              | sequence | redmine
 public | changes                                    | table    | redmine
 public | changes_id_seq                             | sequence | redmine
 public | changeset_parents                          | table    | redmine
 public | changesets                                 | table    | redmine
 public | changesets_id_seq                          | sequence | redmine
 public | changesets_issues                          | table    | redmine
 public | comments                                   | table    | redmine
 public | comments_id_seq                            | sequence | redmine
 public | custom_field_enumerations                  | table    | redmine
 public | custom_field_enumerations_id_seq           | sequence | redmine
 public | custom_fields                              | table    | redmine
 public | custom_fields_id_seq                       | sequence | redmine
 public | custom_fields_projects                     | table    | redmine
 public | custom_fields_roles                        | table    | redmine
 public | custom_fields_trackers                     | table    | redmine
 public | custom_values                              | table    | redmine
 public | custom_values_id_seq                       | sequence | redmine
 public | documents                                  | table    | redmine
 public | documents_id_seq                           | sequence | redmine
 public | email_addresses                            | table    | redmine
 public | email_addresses_id_seq                     | sequence | redmine
 public | enabled_modules                            | table    | redmine
 public | enabled_modules_id_seq                     | sequence | redmine
 public | enumerations                               | table    | redmine
 public | enumerations_id_seq                        | sequence | redmine
 public | groups_users                               | table    | redmine
 public | import_items                               | table    | redmine
 public | import_items_id_seq                        | sequence | redmine
 public | imports                                    | table    | redmine
 public | imports_id_seq                             | sequence | redmine
 public | issue_categories                           | table    | redmine
 public | issue_categories_id_seq                    | sequence | redmine
 public | issue_relations                            | table    | redmine
 public | issue_relations_id_seq                     | sequence | redmine
 public | issue_statuses                             | table    | redmine
 public | issue_statuses_id_seq                      | sequence | redmine
 public | issues                                     | table    | redmine
 public | issues_id_seq                              | sequence | redmine
 public | journal_details                            | table    | redmine
 public | journal_details_id_seq                     | sequence | redmine
 public | journals                                   | table    | redmine
 public | journals_id_seq                            | sequence | redmine
 public | member_roles                               | table    | redmine
 public | member_roles_id_seq                        | sequence | redmine
 public | members                                    | table    | redmine
 public | members_id_seq                             | sequence | redmine
 public | messages                                   | table    | redmine
 public | messages_id_seq                            | sequence | redmine
 public | news                                       | table    | redmine
 public | news_id_seq                                | sequence | redmine
 public | open_id_authentication_associations        | table    | redmine
 public | open_id_authentication_associations_id_seq | sequence | redmine
 public | open_id_authentication_nonces              | table    | redmine
 public | open_id_authentication_nonces_id_seq       | sequence | redmine
 public | projects                                   | table    | redmine
 public | projects_id_seq                            | sequence | redmine
 public | projects_trackers                          | table    | redmine
 public | queries                                    | table    | redmine
 public | queries_id_seq                             | sequence | redmine
 public | queries_roles                              | table    | redmine
 public | repositories                               | table    | redmine
 public | repositories_id_seq                        | sequence | redmine
 public | roles                                      | table    | redmine
 public | roles_id_seq                               | sequence | redmine
 public | roles_managed_roles                        | table    | redmine
 public | schema_migrations                          | table    | redmine
 public | settings                                   | table    | redmine
 public | settings_id_seq                            | sequence | redmine
 public | time_entries                               | table    | redmine
 public | time_entries_id_seq                        | sequence | redmine
 public | tokens                                     | table    | redmine
 public | tokens_id_seq                              | sequence | redmine
 public | trackers                                   | table    | redmine
 public | trackers_id_seq                            | sequence | redmine
 public | user_preferences                           | table    | redmine
 public | user_preferences_id_seq                    | sequence | redmine
 public | users                                      | table    | redmine
 public | users_id_seq                               | sequence | redmine
 public | versions                                   | table    | redmine
 public | versions_id_seq                            | sequence | redmine
 public | watchers                                   | table    | redmine
 public | watchers_id_seq                            | sequence | redmine
 public | wiki_content_versions                      | table    | redmine
 public | wiki_content_versions_id_seq               | sequence | redmine
 public | wiki_contents                              | table    | redmine
 public | wiki_contents_id_seq                       | sequence | redmine
 public | wiki_pages                                 | table    | redmine
 public | wiki_pages_id_seq                          | sequence | redmine
 public | wiki_redirects                             | table    | redmine
 public | wiki_redirects_id_seq                      | sequence | redmine
 public | wikis                                      | table    | redmine
 public | wikis_id_seq                               | sequence | redmine
 public | workflows                                  | table    | redmine
 public | workflows_id_seq                           | sequence | redmine
(101 rows)

redmine=> 
~~~

## 文件权限的配置

运行应用的用户必须对下面的一些子目录有写权限


* files (storage of attachments)
* log (application log file production.log)
* tmp and tmp/pdf (create these ones if not present, used to generate PDF documents among other things)
* public/plugin_assets (assets of plugins)

如果使用 redmine 用户来运行应用的话，可以按如下方式操作

~~~
mkdir -p tmp tmp/pdf public/plugin_assets
sudo chown -R redmine:redmine files log tmp public/plugin_assets
sudo chmod -R 755 files log tmp public/plugin_assets
~~~

如果有文件(备份恢复文档)在如下目录，要确保清掉可执行权限(为了安全)

~~~
sudo find files log tmp public/plugin_assets -type f -exec chmod -x {} +
~~~

## 测试安装

通过 **WEBrick** 来运行安装

~~~
[root@redmine redmine-4.0.4]# bundle exec rails server webrick -e production
=> Booting WEBrick
=> Rails 5.2.3 application starting in production on http://0.0.0.0:3000
=> Run `rails server -h` for more startup options
[2019-08-25 06:01:15] INFO  WEBrick 1.4.2
[2019-08-25 06:01:15] INFO  ruby 2.6.3 (2019-04-16) [x86_64-linux]
[2019-08-25 06:01:15] INFO  WEBrick::HTTPServer#start: pid=8174 port=3000
．．．
．．．
．．．
~~~


进行访问

http://10.1.0.201:3000/

默认密码为:

* login: admin
* password: admin


![redmine](/assets/img/redmine/redmine01.png)

**Webrick** 只是用来作为测试，不适合生产环境，生产环境下适合其它的方式，如 **Unicorn, Thin, Puma, hellip**


>**Note:** Webrick is not suitable for production use, please only use webrick for testing that the installation up to this point is functional. Use one of the many other guides in this wiki to setup redmine to use either Passenger (aka mod_rails), FCGI or a Rack server (Unicorn, Thin, Puma, hellip;) to serve up your redmine.

## 登录

登录界面

![redmine](/assets/img/redmine/redmine02.png)

输入账号密码

![redmine](/assets/img/redmine/redmine03.png)

登录后提示修改密码

![redmine](/assets/img/redmine/redmine04.png)

密码要满足一定长度

![redmine](/assets/img/redmine/redmine05.png)

![redmine](/assets/img/redmine/redmine06.png)

修改完成后，进入个人设置界面

![redmine](/assets/img/redmine/redmine07.png)

日历，甘特图都有，可以方便地进行事务管理

![redmine](/assets/img/redmine/redmine08.png)

![redmine](/assets/img/redmine/redmine09.png)



---

# 总结

一个敏捷开发团队一定会需要一套项目管理软件

以下评价来源于百度百科

* Trac: 基于 Python 的开源程序，应该是最早将 Ticket 与项目结合起来的开发管理系统，支持 Wiki、Timeline、Report 和项目模块分级与里程碑定义，还能够绑定查看SVN内容，简单易用，但是团队开发速度太慢，很多功能缺失，无法进行权限分配、多项目管理，配置不够灵活
* Jira+Confluence: 基于 Java 的 Bug 追踪和企业 Wiki 系统，需要购买，而且很贵，Jira 的 Bug 和事务流管理能力很强大，Confluence 应该是目前最好的企业 Wiki 系统，扩展性强
* ActiveCollab: 基于 PHP 的 Web 项目管理程序，曾经是开源版本的，后来给商业化了，需要购买，Trac 与 Basecamp 的模仿者，安装和使用简单
* 其它: 还有许多 SaaS 方式的在线项目管理服务，例如：Comindwork、LiquidPlanner 、MyQuire、ProjectSpaces、Huddle、PlanHQ、Goplan 等，不过介于中国的出口带宽情况和用户心态问题，将重要的项目数据放在遥远的第三方当前来说还是有些不现实


Redmine是 Trac + Basecamp 的混合体，吸取了两个系统的优点，基于 Ruby on Rails 框架开发，开放源代码，可以跨平台部署

所以 **Redmine** 不失为创业公司初期的一个好选择

同时对于项目敏捷管理，也提供了一个开源的选择和可能



* TOC
{:toc}


---


[redmine]:https://www.redmine.org/
[redmine_install]:https://www.redmine.org/projects/redmine/wiki/RedmineInstallnothing@pc:~/vagrant/blog_c7x64/biscuits/_posts$ 
