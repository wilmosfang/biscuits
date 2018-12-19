---
layout: post
title: "Install PostgreSQL 11"
date: 2018-12-19 17:39:59
image: '/assets/img/'
description: '安装 PostgreSQL 11'
main-class:  postgresql
color: '#336790'
tags:
 - postgresql
categories:
 - postgresql
twitter_text: 'simple process of PostgreSQL installation'
introduction: 'installation of PostgreSQL'
---

## 前言

**[PostgreSQL][postgresql]** 号称是这个世界上最高级的开源数据库

**[PostgreSQL][postgresql]** 的影响力越来越大了，虽然长期居于数据库排行榜的第四名(前三分别为 oracle, mysql, sqlserver)，不过近三年来(2015-2018年)，却是受关注涨幅最大的数据库，并且长期保持稳步增涨的态势，可能与其丰富的特性迎合了现代互联网的发展需求有一定关联

这里分享一下 **[PostgreSQL][postgresql]** 的安装方法

参考 **[Linux downloads (Red Hat family)][postgresql_dl]**


> **Tip:** 当前的版本为 **Version 11.1**

---

# 操作


## 环境

~~~
[vagrant@go ~]$ hostnamectl 
   Static hostname: go
         Icon name: computer-vm
           Chassis: vm
        Machine ID: b6191cc6682d4a308ad289f7f26c4849
           Boot ID: 892566cc1ce94522881e2b28028bede6
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.1.3.el7.x86_64
      Architecture: x86-64
[vagrant@go ~]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:84:81:d5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 84047sec preferred_lft 84047sec
    inet6 fe80::5054:ff:fe84:81d5/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:4f:40:e0 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.101/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe4f:40e0/64 scope link 
       valid_lft forever preferred_lft forever
[vagrant@go ~]$ 
~~~


## 安装仓库

~~~
[root@go golang_learning]# yum install https://download.postgresql.org/pub/repos/yum/11/redhat/rhel-7-x86_64/pgdg-redhat11-11-2.noarch.rpm
Loaded plugins: fastestmirror
pgdg-redhat11-11-2.noarch.rpm                            | 4.8 kB     00:00     
Examining /var/tmp/yum-root-zmy4wk/pgdg-redhat11-11-2.noarch.rpm: pgdg-redhat11-11-2.noarch
Marking /var/tmp/yum-root-zmy4wk/pgdg-redhat11-11-2.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package pgdg-redhat11.noarch 0:11-2 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package            Arch        Version   Repository                       Size
================================================================================
Installing:
 pgdg-redhat11      noarch      11-2      /pgdg-redhat11-11-2.noarch      2.7 k

Transaction Summary
================================================================================
Install  1 Package

Total size: 2.7 k
Installed size: 2.7 k
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : pgdg-redhat11-11-2.noarch                                    1/1 
  Verifying  : pgdg-redhat11-11-2.noarch                                    1/1 

Installed:
  pgdg-redhat11.noarch 0:11-2                                                   

Complete!
[root@go golang_learning]#
~~~


## 安装客户端

~~~
[root@go golang_learning]#  yum install postgresql11
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
pgdg11                                                   | 4.1 kB     00:00     
(1/2): pgdg11/7/x86_64/group_gz                            |  245 B   00:01     
(2/2): pgdg11/7/x86_64/primary_db                          | 137 kB   00:02     
Resolving Dependencies
--> Running transaction check
---> Package postgresql11.x86_64 0:11.1-1PGDG.rhel7 will be installed
--> Processing Dependency: postgresql11-libs(x86-64) = 11.1-1PGDG.rhel7 for package: postgresql11-11.1-1PGDG.rhel7.x86_64
--> Processing Dependency: libpq.so.5()(64bit) for package: postgresql11-11.1-1PGDG.rhel7.x86_64
--> Running transaction check
---> Package postgresql11-libs.x86_64 0:11.1-1PGDG.rhel7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                 Arch         Version                Repository    Size
================================================================================
Installing:
 postgresql11            x86_64       11.1-1PGDG.rhel7       pgdg11       1.6 M
Installing for dependencies:
 postgresql11-libs       x86_64       11.1-1PGDG.rhel7       pgdg11       359 k

Transaction Summary
================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 2.0 M
Installed size: 10 M
Is this ok [y/d/N]: y
Downloading packages:
(1/2): postgresql11-libs-11.1-1PGDG.rhel7.x86_64.rpm       | 359 kB   00:06     
(2/2): postgresql11-11.1-1PGDG.rhel7.x86_64.rpm            | 1.6 MB   00:09     
--------------------------------------------------------------------------------
Total                                              206 kB/s | 2.0 MB  00:09     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : postgresql11-libs-11.1-1PGDG.rhel7.x86_64                    1/2 
  Installing : postgresql11-11.1-1PGDG.rhel7.x86_64                         2/2 
  Verifying  : postgresql11-11.1-1PGDG.rhel7.x86_64                         1/2 
  Verifying  : postgresql11-libs-11.1-1PGDG.rhel7.x86_64                    2/2 

Installed:
  postgresql11.x86_64 0:11.1-1PGDG.rhel7                                        

Dependency Installed:
  postgresql11-libs.x86_64 0:11.1-1PGDG.rhel7                                   

Complete!
[root@go golang_learning]# 
~~~



## 安装服务端

~~~
[root@go golang_learning]# yum install postgresql11-server
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.pregi.net
 * extras: mirror.pregi.net
 * updates: mirror.pregi.net
Resolving Dependencies
--> Running transaction check
---> Package postgresql11-server.x86_64 0:11.1-1PGDG.rhel7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package                  Arch        Version                 Repository   Size
================================================================================
Installing:
 postgresql11-server      x86_64      11.1-1PGDG.rhel7        pgdg11      4.7 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 4.7 M
Installed size: 19 M
Is this ok [y/d/N]: y
Downloading packages:
postgresql11-server-11.1-1PGDG.rhel7.x86_64.rpm            | 4.7 MB   00:19     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : postgresql11-server-11.1-1PGDG.rhel7.x86_64                  1/1 
  Verifying  : postgresql11-server-11.1-1PGDG.rhel7.x86_64                  1/1 

Installed:
  postgresql11-server.x86_64 0:11.1-1PGDG.rhel7                                 

Complete!
[root@go golang_learning]#
~~~


## 初始化数据库

~~~
[root@go golang_learning]# /usr/pgsql-11/bin/postgresql-11-setup initdb
Initializing database ... OK

[root@go golang_learning]# 
~~~


## 启动服务

~~~
[root@go golang_learning]# systemctl enable postgresql-11
Created symlink from /etc/systemd/system/multi-user.target.wants/postgresql-11.service to /usr/lib/systemd/system/postgresql-11.service.
[root@go golang_learning]# systemctl start postgresql-11
[root@go golang_learning]#
~~~


## 访问数据库


~~~
[root@go ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
adm:x:3:4:adm:/var/adm:/sbin/nologin
lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
sync:x:5:0:sync:/sbin:/bin/sync
shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
halt:x:7:0:halt:/sbin:/sbin/halt
mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
operator:x:11:0:operator:/root:/sbin/nologin
games:x:12:100:games:/usr/games:/sbin/nologin
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
nobody:x:99:99:Nobody:/:/sbin/nologin
systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
dbus:x:81:81:System message bus:/:/sbin/nologin
polkitd:x:999:998:User for polkitd:/:/sbin/nologin
rpc:x:32:32:Rpcbind Daemon:/var/lib/rpcbind:/sbin/nologin
rpcuser:x:29:29:RPC Service User:/var/lib/nfs:/sbin/nologin
nfsnobody:x:65534:65534:Anonymous NFS User:/var/lib/nfs:/sbin/nologin
sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
postfix:x:89:89::/var/spool/postfix:/sbin/nologin
chrony:x:998:995::/var/lib/chrony:/sbin/nologin
vagrant:x:1000:1000:vagrant:/home/vagrant:/bin/bash
postgres:x:26:26:PostgreSQL Server:/var/lib/pgsql:/bin/bash
[root@go ~]# 
[root@go ~]# su - postres
su: user postres does not exist
[root@go ~]# su - postgres
-bash-4.2$ cd /var/lib/pgsql/11/data/
-bash-4.2$ ls
base              pg_ident.conf  pg_stat      pg_xact
current_logfiles  pg_logical     pg_stat_tmp  postgresql.auto.conf
global            pg_multixact   pg_subtrans  postgresql.conf
log               pg_notify      pg_tblspc    postmaster.opts
pg_commit_ts      pg_replslot    pg_twophase  postmaster.pid
pg_dynshmem       pg_serial      PG_VERSION
pg_hba.conf       pg_snapshots   pg_wal
-bash-4.2$ createdb testdb
-bash-4.2$ ls
base              pg_ident.conf  pg_stat      pg_xact
current_logfiles  pg_logical     pg_stat_tmp  postgresql.auto.conf
global            pg_multixact   pg_subtrans  postgresql.conf
log               pg_notify      pg_tblspc    postmaster.opts
pg_commit_ts      pg_replslot    pg_twophase  postmaster.pid
pg_dynshmem       pg_serial      PG_VERSION
pg_hba.conf       pg_snapshots   pg_wal
-bash-4.2$ psql testdb
psql (11.1)
Type "help" for help.

testdb=# select version();
                                                 version                        
                         
--------------------------------------------------------------------------------
-------------------------
 PostgreSQL 11.1 on x86_64-pc-linux-gnu, compiled by gcc (GCC) 4.8.5 20150623 (R
ed Hat 4.8.5-28), 64-bit
(1 row)

testdb=# \h
Available help:
  ABORT                            CREATE USER
  ALTER AGGREGATE                  CREATE USER MAPPING
  ALTER COLLATION                  CREATE VIEW
  ALTER CONVERSION                 DEALLOCATE
  ALTER DATABASE                   DECLARE
  ALTER DEFAULT PRIVILEGES         DELETE
  ALTER DOMAIN                     DISCARD
  ALTER EVENT TRIGGER              DO
  ALTER EXTENSION                  DROP ACCESS METHOD
  ALTER FOREIGN DATA WRAPPER       DROP AGGREGATE
  ALTER FOREIGN TABLE              DROP CAST
  ALTER FUNCTION                   DROP COLLATION
  ALTER GROUP                      DROP CONVERSION
  ALTER INDEX                      DROP DATABASE
  ALTER LANGUAGE                   DROP DOMAIN
  ALTER LARGE OBJECT               DROP EVENT TRIGGER
  ALTER MATERIALIZED VIEW          DROP EXTENSION
  ALTER OPERATOR                   DROP FOREIGN DATA WRAPPER
  ALTER OPERATOR CLASS             DROP FOREIGN TABLE
  ALTER OPERATOR FAMILY            DROP FUNCTION
  ALTER POLICY                     DROP GROUP
  ALTER PROCEDURE                  DROP INDEX
  ALTER PUBLICATION                DROP LANGUAGE
  ALTER ROLE                       DROP MATERIALIZED VIEW
  ALTER ROUTINE                    DROP OPERATOR
  ALTER RULE                       DROP OPERATOR CLASS
  ALTER SCHEMA                     DROP OPERATOR FAMILY
  ALTER SEQUENCE                   DROP OWNED
  ALTER SERVER                     DROP POLICY
  ALTER STATISTICS                 DROP PROCEDURE
  ALTER SUBSCRIPTION               DROP PUBLICATION
  ALTER SYSTEM                     DROP ROLE
  ALTER TABLE                      DROP ROUTINE
  ALTER TABLESPACE                 DROP RULE
  ALTER TEXT SEARCH CONFIGURATION  DROP SCHEMA
  ALTER TEXT SEARCH DICTIONARY     DROP SEQUENCE
  ALTER TEXT SEARCH PARSER         DROP SERVER
  ALTER TEXT SEARCH TEMPLATE       DROP STATISTICS
  ALTER TRIGGER                    DROP SUBSCRIPTION
  ALTER TYPE                       DROP TABLE
  ALTER USER                       DROP TABLESPACE
  ALTER USER MAPPING               DROP TEXT SEARCH CONFIGURATION
  ALTER VIEW                       DROP TEXT SEARCH DICTIONARY
  ANALYZE                          DROP TEXT SEARCH PARSER
  BEGIN                            DROP TEXT SEARCH TEMPLATE
  CALL                             DROP TRANSFORM
  CHECKPOINT                       DROP TRIGGER
  CLOSE                            DROP TYPE
  CLUSTER                          DROP USER
  COMMENT                          DROP USER MAPPING
  COMMIT                           DROP VIEW
  COMMIT PREPARED                  END
  COPY                             EXECUTE
  CREATE ACCESS METHOD             EXPLAIN
  CREATE AGGREGATE                 FETCH
  CREATE CAST                      GRANT
  CREATE COLLATION                 IMPORT FOREIGN SCHEMA
  CREATE CONVERSION                INSERT
  CREATE DATABASE                  LISTEN
  CREATE DOMAIN                    LOAD
  CREATE EVENT TRIGGER             LOCK
  CREATE EXTENSION                 MOVE
  CREATE FOREIGN DATA WRAPPER      NOTIFY
  CREATE FOREIGN TABLE             PREPARE
  CREATE FUNCTION                  PREPARE TRANSACTION
  CREATE GROUP                     REASSIGN OWNED
  CREATE INDEX                     REFRESH MATERIALIZED VIEW
  CREATE LANGUAGE                  REINDEX
  CREATE MATERIALIZED VIEW         RELEASE SAVEPOINT
  CREATE OPERATOR                  RESET
  CREATE OPERATOR CLASS            REVOKE
  CREATE OPERATOR FAMILY           ROLLBACK
  CREATE POLICY                    ROLLBACK PREPARED
  CREATE PROCEDURE                 ROLLBACK TO SAVEPOINT
  CREATE PUBLICATION               SAVEPOINT
  CREATE ROLE                      SECURITY LABEL
  CREATE RULE                      SELECT
  CREATE SCHEMA                    SELECT INTO
  CREATE SEQUENCE                  SET
  CREATE SERVER                    SET CONSTRAINTS
  CREATE STATISTICS                SET ROLE
  CREATE SUBSCRIPTION              SET SESSION AUTHORIZATION
  CREATE TABLE                     SET TRANSACTION
  CREATE TABLE AS                  SHOW
  CREATE TABLESPACE                START TRANSACTION
  CREATE TEXT SEARCH CONFIGURATION TABLE
  CREATE TEXT SEARCH DICTIONARY    TRUNCATE
  CREATE TEXT SEARCH PARSER        UNLISTEN
testdb=# \q
-bash-4.2$ 
~~~



---

# 总结

由于 **[PostgreSQL][postgresql]** 准备好了相应的 yum 仓库，所以整个过程就是一个 rpm 包的安装过程

关于 PostgreSQL 的配置，后面有需要，再进行展开

* TOC
{:toc}

---

[postgresql]:https://www.postgresql.org/
[postgresql_dl]:https://www.postgresql.org/download/linux/redhat/
