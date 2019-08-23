---
layout: post
title: "Backing up and Restoring in InfluxDB"
date: 2019-08-23 02:22:28
image: '/assets/img/'
description: '备份恢复 InfluxDB'
main-class:  influxdb
color: '#4591ed'
tags:
 - influxdb
categories:
 - influxdb
twitter_text: 'simple process of backing up and restoring in influxdb'
introduction: 'simple process of backing up and restoring in influxdb'
---


## 前言

**[InfluxDB][influxdb]** 是一款高性能的时序数据库

>InfluxDB is a time series database built from the ground up to handle high write and query loads. It is the second piece of the TICK stack. InfluxDB is meant to be used as a backing store for any use case involving large amounts of timestamped data, including DevOps monitoring, application metrics, IoT sensor data, and real-time analytics

**[InfluxDB][influxdb]** 见长于标量的时序存储

当前最主要的应用场景有以下两个方面

* 监控数据
* IoT 数据流存储

这两方面的特性 **Elasticsearch** 也有覆盖，那它们两者的区别是什么呢，可以参考下面的文章

**[InfluxDB vs. Elasticsearch for Time Series Data & Metrics Benchmark][Influxdb_vs_es]**

总体来讲，在日志与全文检索方面 **Elasticsearch** 还是无可匹敌的，但是在纯标量的存储检索压缩和响应方面 **InfluxDB** 会有更明显的优势，所以事实上各有侧重

同时，类似于 **Elasticsearch** 的 **ELK** 技术栈，**InfluxDB** 也有一套 **[TICK][tick]** 技术栈

这里分享一下 **[InfluxDB][influxdb]** 备份恢复的方法

参考 **[Backing up and restoring in InfluxDB][backup_and_restore]**

> **Tip:** 当前的版本为 **InfluxDB v1.7.7 (git: 1.7 f8fdf652f348fc9980997fe1c972e2b79ddd13b0)**

---

# 操作


## 环境

~~~
[root@influxdb ~]# hostnamectl 
   Static hostname: influxdb
         Icon name: computer-vm
           Chassis: vm
        Machine ID: a8128cfbdef948119ae7fb0f88b37ee7
           Boot ID: f8fd112df4eb43a6905c7066c1d57c2e
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.1.3.el7.x86_64
      Architecture: x86-64
[root@influxdb ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:84:81:d5 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 86040sec preferred_lft 86040sec
    inet6 fe80::5054:ff:fe84:81d5/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:a2:bc:94 brd ff:ff:ff:ff:ff:ff
    inet 10.1.0.51/24 brd 10.1.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fea2:bc94/64 scope link 
       valid_lft forever preferred_lft forever
[root@influxdb ~]# influxd version
InfluxDB v1.7.7 (git: 1.7 f8fdf652f348fc9980997fe1c972e2b79ddd13b0)
[root@influxdb ~]# 
~~~


## 更新

从 1.5 的版本开始，InfluxDB 的 **`backup`** 工具就提供了如下一些特性

* **backup** 和 **restore** 可以在线完成备份恢复任务
* 可以针对单个或多个数据库进行基于时间的过滤
* 可以从企业版集群中导入数据
* 可以导出数据到企业版中


## 说明

备份恢复文件只能在小版本号中共享，比如 **backup from 1.7.3 and restore on 1.7.7**

>**Note:** Use the backup and restore utilities to back up and restore between influxd instances with the same versions or with only minor version differences. For example, you can backup from 1.7.3 and restore on 1.7.7.


## 配置远程连接

将 **`bind-address`** 改成一个其它机器可以访问的地址

~~~
[root@influxdb ~]# vim /etc/influxdb/influxdb.conf 
[root@influxdb ~]# cat /etc/influxdb/influxdb.conf | grep -v "#"| cat -s 

bind-address = "10.1.0.51:8088"

[meta]
  dir = "/var/lib/influxdb/meta"

[data]
  dir = "/var/lib/influxdb/data"

  wal-dir = "/var/lib/influxdb/wal"

  series-id-set-cache-size = 100

[coordinator]

[retention]

[shard-precreation]

[monitor]

[http]

[logging]

[subscriber]

[[graphite]]

[[collectd]]

[[opentsdb]]

[[udp]]

[continuous_queries]

[tls]

[root@influxdb ~]# 
[root@influxdb ~]# grep -C 2 127.0.0.1:8088  /etc/influxdb/influxdb.conf 

# Bind address to use for the RPC service for backup and restore.
# bind-address = "127.0.0.1:8088"
bind-address = "10.1.0.51:8088"

[root@influxdb ~]#
[root@influxdb ~]# netstat -ant  | grep 8088
tcp        0      0 127.0.0.1:8088          0.0.0.0:*               LISTEN     
[root@influxdb ~]#
[root@influxdb ~]# systemctl  restart influxdb.service
[root@influxdb ~]# netstat -ant  | grep 8088
tcp        0      0 10.1.0.51:8088          0.0.0.0:*               LISTEN     
[root@influxdb ~]#
~~~

通过网络对目标库进行备份

~~~
[root@influxdb ~]# influxd backup -portable -database new_test -host 10.1.0.51:8088 /tmp/new_test.infuxdb
2019/08/23 05:30:08 backing up metastore to /tmp/new_test.infuxdb/meta.00
2019/08/23 05:30:08 backing up db=new_test
2019/08/23 05:30:08 backing up db=new_test rp=autogen shard=7 to /tmp/new_test.infuxdb/new_test.autogen.00007.00 since 0001-01-01T00:00:00Z
2019/08/23 05:30:08 backing up db=new_test rp=autogen shard=8 to /tmp/new_test.infuxdb/new_test.autogen.00008.00 since 0001-01-01T00:00:00Z
2019/08/23 05:30:08 backup complete:
2019/08/23 05:30:08 	/tmp/new_test.infuxdb/20190823T053008Z.meta
2019/08/23 05:30:08 	/tmp/new_test.infuxdb/20190823T053008Z.s7.tar.gz
2019/08/23 05:30:08 	/tmp/new_test.infuxdb/20190823T053008Z.s8.tar.gz
2019/08/23 05:30:08 	/tmp/new_test.infuxdb/20190823T053008Z.manifest
[root@influxdb ~]# echo $?
0
[root@influxdb ~]# 
[root@influxdb ~]# tree -L 3 /tmp/new_test.infuxdb/
/tmp/new_test.infuxdb/
├── 20190823T053008Z.manifest
├── 20190823T053008Z.meta
├── 20190823T053008Z.s7.tar.gz
└── 20190823T053008Z.s8.tar.gz

0 directories, 4 files
[root@influxdb ~]# 
~~~

## backup

**`backup`** 是 **`influxd`** 的子命令

~~~
[root@influxdb ~]# influxd help
Configure and start an InfluxDB server.

Usage: influxd [[command] [arguments]]

The commands are:

    backup               downloads a snapshot of a data node and saves it to disk
    config               display the default configuration
    help                 display this help message
    restore              uses a snapshot of a data node to rebuild a cluster
    run                  run node with existing configuration
    version              displays the InfluxDB version

"run" is the default command.

Use "influxd [command] -help" for more information about a command.
[root@influxdb ~]# 
~~~

我们看看 **`backup`** 有哪些选项

~~~
[root@influxdb ~]# influxd  backup --help 

Creates a backup copy of specified InfluxDB OSS database(s) and saves the files in an Enterprise-compatible
format to PATH (directory where backups are saved). 

Usage: influxd backup [options] PATH

    -portable
            Required to generate backup files in a portable format that can be restored to InfluxDB OSS or InfluxDB 
            Enterprise. Use unless the legacy backup is required.
    -host <host:port>
            InfluxDB OSS host to back up from. Optional. Defaults to 127.0.0.1:8088.
    -db <name>
            InfluxDB OSS database name to back up. Optional. If not specified, all databases are backed up when 
            using '-portable'.
    -rp <name>
            Retention policy to use for the backup. Optional. If not specified, all retention policies are used by 
            default.
    -shard <id>
            The identifier of the shard to back up. Optional. If specified, '-rp <rp_name>' is required.
    -start <2015-12-24T08:12:23Z>
            Include all points starting with specified timestamp (RFC3339 format). 
            Not compatible with '-since <timestamp>'.
    -end <2015-12-24T08:12:23Z>
            Exclude all points after timestamp (RFC3339 format). 
            Not compatible with '-since <timestamp>'.
    -since <2015-12-24T08:12:23Z>
            Create an incremental backup of all points after the timestamp (RFC3339 format). Optional. 
            Recommend using '-start <timestamp>' instead.
    -skip-errors 
            Optional flag to continue backing up the remaining shards when the current shard fails to backup. 
backup: flag: help requested
[root@influxdb ~]# 
~~~

**`-portable`** 以企业版兼容的方式进行导出



### 备份所有数据库

~~~
[root@influxdb ~]# influxd backup -portable /tmp/influxdb_all
2019/08/23 05:55:28 backing up metastore to /tmp/influxdb_all/meta.00
2019/08/23 05:55:28 No database, retention policy or shard ID given. Full meta store backed up.
2019/08/23 05:55:28 Backing up all databases in portable format
2019/08/23 05:55:28 backing up db=
2019/08/23 05:55:28 backing up db=_internal rp=monitor shard=16 to /tmp/influxdb_all/_internal.monitor.00016.00 since 0001-01-01T00:00:00Z
2019/08/23 05:55:28 backing up db=grafana_test rp=autogen shard=2 to /tmp/influxdb_all/grafana_test.autogen.00002.00 since 0001-01-01T00:00:00Z
2019/08/23 05:55:28 backing up db=grafana_test rp=autogen shard=3 to /tmp/influxdb_all/grafana_test.autogen.00003.00 since 0001-01-01T00:00:00Z
2019/08/23 05:55:28 backing up db=new_test rp=autogen shard=7 to /tmp/influxdb_all/new_test.autogen.00007.00 since 0001-01-01T00:00:00Z
2019/08/23 05:55:28 backing up db=new_test rp=autogen shard=8 to /tmp/influxdb_all/new_test.autogen.00008.00 since 0001-01-01T00:00:00Z
2019/08/23 05:55:28 backup complete:
2019/08/23 05:55:28 	/tmp/influxdb_all/20190823T055528Z.meta
2019/08/23 05:55:28 	/tmp/influxdb_all/20190823T055528Z.s16.tar.gz
2019/08/23 05:55:28 	/tmp/influxdb_all/20190823T055528Z.s2.tar.gz
2019/08/23 05:55:28 	/tmp/influxdb_all/20190823T055528Z.s3.tar.gz
2019/08/23 05:55:28 	/tmp/influxdb_all/20190823T055528Z.s7.tar.gz
2019/08/23 05:55:28 	/tmp/influxdb_all/20190823T055528Z.s8.tar.gz
2019/08/23 05:55:28 	/tmp/influxdb_all/20190823T055528Z.manifest
[root@influxdb ~]# tree -L 3 /tmp/influxdb_all/
/tmp/influxdb_all/
├── 20190823T055528Z.manifest
├── 20190823T055528Z.meta
├── 20190823T055528Z.s16.tar.gz
├── 20190823T055528Z.s2.tar.gz
├── 20190823T055528Z.s3.tar.gz
├── 20190823T055528Z.s7.tar.gz
└── 20190823T055528Z.s8.tar.gz

0 directories, 7 files
[root@influxdb ~]#
~~~


### 指定开始时间的备份


~~~
[root@influxdb ~]# influxd backup -portable  -start 2017-04-28T06:49:00Z  /tmp/influxdb_all.start
2019/08/23 08:50:28 backing up metastore to /tmp/influxdb_all.start/meta.00
2019/08/23 08:50:28 No database, retention policy or shard ID given. Full meta store backed up.
2019/08/23 08:50:28 Backing up all databases in portable format
2019/08/23 08:50:28 backing up db=
2019/08/23 08:50:28 backing up db=_internal rp=monitor shard=16 to /tmp/influxdb_all.start/_internal.monitor.00016.00 with boundaries start=2017-04-28T06:49:00Z, end=0001-01-01T00:00:00Z
2019/08/23 08:50:28 backing up db=grafana_test rp=autogen shard=2 to /tmp/influxdb_all.start/grafana_test.autogen.00002.00 with boundaries start=2017-04-28T06:49:00Z, end=0001-01-01T00:00:00Z
2019/08/23 08:50:28 backing up db=grafana_test rp=autogen shard=3 to /tmp/influxdb_all.start/grafana_test.autogen.00003.00 with boundaries start=2017-04-28T06:49:00Z, end=0001-01-01T00:00:00Z
2019/08/23 08:50:28 backing up db=new_test rp=autogen shard=7 to /tmp/influxdb_all.start/new_test.autogen.00007.00 with boundaries start=2017-04-28T06:49:00Z, end=0001-01-01T00:00:00Z
2019/08/23 08:50:28 backing up db=new_test rp=autogen shard=8 to /tmp/influxdb_all.start/new_test.autogen.00008.00 with boundaries start=2017-04-28T06:49:00Z, end=0001-01-01T00:00:00Z
2019/08/23 08:50:28 backup complete:
2019/08/23 08:50:28 	/tmp/influxdb_all.start/20190823T085028Z.meta
2019/08/23 08:50:28 	/tmp/influxdb_all.start/20190823T085028Z.s16.tar.gz
2019/08/23 08:50:28 	/tmp/influxdb_all.start/20190823T085028Z.s2.tar.gz
2019/08/23 08:50:28 	/tmp/influxdb_all.start/20190823T085028Z.s3.tar.gz
2019/08/23 08:50:28 	/tmp/influxdb_all.start/20190823T085028Z.s7.tar.gz
2019/08/23 08:50:28 	/tmp/influxdb_all.start/20190823T085028Z.s8.tar.gz
2019/08/23 08:50:28 	/tmp/influxdb_all.start/20190823T085028Z.manifest
[root@influxdb ~]# 
~~~

### 指定数据库备份

~~~
[root@influxdb ~]# influxd backup -portable  -database new_test  /tmp/influxdb.new_test
2019/08/23 10:57:00 backing up metastore to /tmp/influxdb.new_test/meta.00
2019/08/23 10:57:00 backing up db=new_test
2019/08/23 10:57:00 backing up db=new_test rp=autogen shard=7 to /tmp/influxdb.new_test/new_test.autogen.00007.00 since 0001-01-01T00:00:00Z
2019/08/23 10:57:00 backing up db=new_test rp=autogen shard=8 to /tmp/influxdb.new_test/new_test.autogen.00008.00 since 0001-01-01T00:00:00Z
2019/08/23 10:57:00 backup complete:
2019/08/23 10:57:00 	/tmp/influxdb.new_test/20190823T105700Z.meta
2019/08/23 10:57:00 	/tmp/influxdb.new_test/20190823T105700Z.s7.tar.gz
2019/08/23 10:57:00 	/tmp/influxdb.new_test/20190823T105700Z.s8.tar.gz
2019/08/23 10:57:00 	/tmp/influxdb.new_test/20190823T105700Z.manifest
[root@influxdb ~]#
~~~

#### 指定时间范围备份

~~~
[root@influxdb ~]# influxd backup  -portable -database new_test -start 2017-04-28T06:49:00Z -end 2017-04-28T06:50:00Z /tmp/new_test.db
2019/08/23 11:05:29 backing up metastore to /tmp/new_test.db/meta.00
2019/08/23 11:05:29 backing up db=new_test
2019/08/23 11:05:29 backing up db=new_test rp=autogen shard=7 to /tmp/new_test.db/new_test.autogen.00007.00 with boundaries start=2017-04-28T06:49:00Z, end=2017-04-28T06:50:00Z
2019/08/23 11:05:29 backing up db=new_test rp=autogen shard=8 to /tmp/new_test.db/new_test.autogen.00008.00 with boundaries start=2017-04-28T06:49:00Z, end=2017-04-28T06:50:00Z
2019/08/23 11:05:29 backup complete:
2019/08/23 11:05:29 	/tmp/new_test.db/20190823T110529Z.meta
2019/08/23 11:05:29 	/tmp/new_test.db/20190823T110529Z.s7.tar.gz
2019/08/23 11:05:29 	/tmp/new_test.db/20190823T110529Z.s8.tar.gz
2019/08/23 11:05:29 	/tmp/new_test.db/20190823T110529Z.manifest
[root@influxdb ~]# 
~~~


## restore

~~~
[root@influxdb ~]# influxd restore --help 

Uses backup copies from the specified PATH to restore databases or specific shards from InfluxDB OSS
  or InfluxDB Enterprise to an InfluxDB OSS instance.

Usage: influxd restore -portable [options] PATH

Note: Restore using the '-portable' option consumes files in an improved Enterprise-compatible 
  format that includes a file manifest.

Options:
    -portable 
            Required to activate the portable restore mode. If not specified, the legacy restore mode is used.
    -host  <host:port>
            InfluxDB OSS host to connect to where the data will be restored. Defaults to '127.0.0.1:8088'.
    -db    <name>
            Name of database to be restored from the backup (InfluxDB OSS or InfluxDB Enterprise)
    -newdb <name>
            Name of the InfluxDB OSS database into which the archived data will be imported on the target system. 
            Optional. If not given, then the value of '-db <db_name>' is used.  The new database name must be unique 
            to the target system.
    -rp    <name>
            Name of retention policy from the backup that will be restored. Optional. 
            Requires that '-db <db_name>' is specified.
    -newrp <name>
            Name of the retention policy to be created on the target system. Optional. Requires that '-rp <rp_name>' 
            is set. If not given, the '-rp <rp_name>' value is used.
    -shard <id>
            Identifier of the shard to be restored. Optional. If specified, then '-db <db_name>' and '-rp <rp_name>' are
            required.
    PATH
            Path to directory containing the backup files.

restore: flag: help requested
[root@influxdb ~]# 
~~~


**`-newdb`** 是用来指定在新的数据库中叫什么名字, 其它加 **`new`** 前缀的意义一样

### 恢复所有数据

先清除本地数据

~~~
[root@influxdb ~]# influx
Connected to http://localhost:8086 version 1.7.7
InfluxDB shell version: 1.7.7
> show databases;
name: databases
name
----
_internal
grafana_test
new_test
> drop database new_test;
> show databases;
name: databases
name
----
_internal
grafana_test
> drop database grafana_test;
> show databases
name: databases
name
----
_internal
> exit
[root@influxdb ~]#
~~~


再导入之前备份的数据

~~~
[root@influxdb ~]# influxd restore -portable  /tmp/influxdb_all
2019/08/23 11:25:01 Meta info not found for shard 16 on database _internal. Skipping shard file 20190823T055528Z.s16.tar.gz
2019/08/23 11:25:01 Restoring shard 2 live from backup 20190823T055528Z.s2.tar.gz
2019/08/23 11:25:01 Restoring shard 3 live from backup 20190823T055528Z.s3.tar.gz
2019/08/23 11:25:01 Restoring shard 7 live from backup 20190823T055528Z.s7.tar.gz
2019/08/23 11:25:01 Restoring shard 8 live from backup 20190823T055528Z.s8.tar.gz
[root@influxdb ~]# influx
Connected to http://localhost:8086 version 1.7.7
InfluxDB shell version: 1.7.7
> show databases;
name: databases
name
----
_internal
grafana_test
new_test
> quit
[root@influxdb ~]#
~~~


### 指定一个数据库进行恢复

~~~
[root@influxdb ~]# influx
Connected to http://localhost:8086 version 1.7.7
InfluxDB shell version: 1.7.7
> show databases;
name: databases
name
----
_internal
grafana_test
new_test
> drop database new_test;
> show databases;
name: databases
name
----
_internal
grafana_test
> exit
[root@influxdb ~]# 
[root@influxdb ~]# influxd restore -portable -db new_test /tmp/influxdb.new_test
2019/08/23 12:34:54 Restoring shard 7 live from backup 20190823T112035Z.s7.tar.gz
2019/08/23 12:34:54 Restoring shard 8 live from backup 20190823T112035Z.s8.tar.gz
[root@influxdb ~]# 
~~~

### 指定一个新的数据库名恢复

~~~
[root@influxdb ~]# influxd restore -portable -db new_test -newdb abc_test /tmp/influxdb.new_test
2019/08/23 12:40:11 Restoring shard 8 live from backup 20190823T112035Z.s8.tar.gz
2019/08/23 12:40:11 Restoring shard 7 live from backup 20190823T112035Z.s7.tar.gz
[root@influxdb ~]#
[root@influxdb ~]# influx
Connected to http://localhost:8086 version 1.7.7
InfluxDB shell version: 1.7.7
> show databases;
name: databases
name
----
_internal
grafana_test
new_test
abc_test
> use abc_test;
Using database abc_test
> select count(*) from data;
name: data
time count_value
---- -----------
0    604800
> exit
[root@influxdb ~]#
~~~


## 向前兼容的备份

在 **1.5** 版本之前, 数据备份是分成两部分的

* metastore
* metrics（data）

**influxdb** 有能力进行指定时间点的数据恢复

所有的数据都是全量备份,不支持增量备份

两种类型的数据 **metastore** 和 **metrics** 

**metastore** 会全部备份

**metrics** 的备份是分开进行的


### metastore

**metastore** 包含了所有数据的管理信息

>The InfluxDB metastore contains internal information about the status of the system, including user information, database and shard metadata, continuous queries, retention policies, and subscriptions. While a node is running, you can create a backup of your instance’s metastore by running the command


~~~
[root@influxdb ~]# influxd backup /tmp/metastore
2019/08/23 13:23:07 backing up metastore to /tmp/metastore/meta.00
2019/08/23 13:23:07 No database, retention policy or shard ID given. Full meta store backed up.
2019/08/23 13:23:07 backup complete:
2019/08/23 13:23:07 	/tmp/metastore/tmp/metastore/meta.00
[root@influxdb ~]# 
~~~

### backup

**metrics** 必须以数据库作为区别来进行分开备份

~~~
[root@influxdb ~]# influxd backup -database new_test /tmp/x_new_test 
2019/08/23 14:19:12 backing up metastore to /tmp/x_new_test/meta.00
2019/08/23 14:19:12 backing up db=new_test
2019/08/23 14:19:12 backing up db=new_test rp=autogen shard=25 to /tmp/x_new_test/new_test.autogen.00025.00 since 0001-01-01T00:00:00Z
2019/08/23 14:19:12 backing up db=new_test rp=autogen shard=26 to /tmp/x_new_test/new_test.autogen.00026.00 since 0001-01-01T00:00:00Z
2019/08/23 14:19:12 backup complete:
2019/08/23 14:19:12 	/tmp/x_new_test/tmp/x_new_test/meta.00
2019/08/23 14:19:12 	/tmp/x_new_test/tmp/x_new_test/new_test.autogen.00025.00
2019/08/23 14:19:12 	/tmp/x_new_test/tmp/x_new_test/new_test.autogen.00026.00
[root@influxdb ~]#
~~~

提取出了如下几个文件

~~~
[root@influxdb ~]# tree -L 3 /tmp/x_new_test/
/tmp/x_new_test/
├── meta.00
├── new_test.autogen.00025.00
└── new_test.autogen.00026.00

0 directories, 3 files
[root@influxdb ~]# 
~~~

删除数据

~~~
[root@influxdb ~]# influx
Connected to http://localhost:8086 version 1.7.7
InfluxDB shell version: 1.7.7
> show databases;
name: databases
name
----
_internal
grafana_test
new_test
abc_test
> drop database new_test;
> show databases;
name: databases
name
----
_internal
grafana_test
abc_test
> exit
[root@influxdb ~]# 
~~~

### restore

~~~
[root@influxdb ~]# influxd restore -database new_test -metadir /var/lib/influxdb/meta  -datadir /var/lib/influxdb/data  /tmp/x_new_test
Using metastore snapshot: /tmp/x_new_test/meta.00
2019/08/23 14:20:47 Restoring offline from backup /tmp/x_new_test/new_test.*
[root@influxdb ~]# echo $?
0
[root@influxdb ~]# 
~~~

虽然提示成功

但是其实没有生效

~~~
[root@influxdb ~]# influx
Connected to http://localhost:8086 version 1.7.7
InfluxDB shell version: 1.7.7
> show databases;
name: databases
name
----
_internal
grafana_test
abc_test
> exit
[root@influxdb ~]# 
~~~

>**Note:** 这里还有一个不起眼的小坑, 如何是 root 执行的,　会导致恢复后的权限不对, 让influx 实例读不到数据


~~~
[root@influxdb ~]# ll /var/lib/influxdb/meta/
total 8
-rw-rw-r--. 1 root     root     537 Aug 23 14:20 meta.db
-rw-r-xr-x. 1 influxdb influxdb   5 Aug 23 14:20 node.json
[root@influxdb ~]# ll /var/lib/influxdb/data/
total 16
drwxr-xr-x. 4 influxdb influxdb 4096 Aug 23 14:16 abc_test
drwx------. 4 influxdb influxdb 4096 Aug 23 12:33 grafana_test
drwx------. 4 influxdb influxdb 4096 Jul 19 03:39 _internal
drwxr-xr-x. 3 root     root     4096 Aug 23 14:20 new_test
[root@influxdb ~]# ll /var/lib/influxdb/data/new_test/
total 4
drwxr-xr-x. 4 root root 4096 Aug 23 14:20 autogen
[root@influxdb ~]# ll /var/lib/influxdb/data/grafana_test/
total 8
drwx------.  4 influxdb influxdb 4096 Aug 23 12:33 autogen
drwxr-xr-x. 10 influxdb influxdb 4096 Aug 23 12:33 _series
[root@influxdb ~]#
~~~


此时贸然重启 influxdb 会让它启不来，需要调整权限


~~~
[root@influxdb ~]# chown -R influxdb.influxdb  /var/lib/influxdb/
[root@influxdb ~]# systemctl  restart influxdb.service 
[root@influxdb ~]# 
~~~

再登入检查数据

~~~
[root@influxdb ~]# influx
Connected to http://localhost:8086 version 1.7.7
InfluxDB shell version: 1.7.7
> show databases
name: databases
name
----
_internal
grafana_test
new_test
abc_test
> use new_test;
Using database new_test
> show measurements
name: measurements
name
----
data
> select count(*) from  data;
name: data
time count_value
---- -----------
0    604800
> exit
[root@influxdb ~]# 
~~~

完成了数据恢复


---

# 总结

**influxdb** 是一个非常简洁的数据库

简单是高效的基础

并且随着大数据的进一步发展，可以预见，此类时序数据库会越来越受到欢迎

这里 **1.5** 版本前后有所变化

**`-portable `** 的方式在线备份和恢复要容易得多

使用老的方式要复杂很多，官方建议恢复过程要停机，从实验中也可以看出来，即便不停机恢复数据也不可见，依然得重启实例，这样在线上就会影响服务的可用性

同时还有权限的相关问题，需要特别注意

暂时不支持增量，所有关于一个库的备份都是全量备份

* TOC
{:toc}


---


[influxdb]:https://portal.influxdata.com/
[backup_and_restore]:https://docs.influxdata.com/influxdb/v1.7/administration/backup_and_restore
[Influxdb_vs_es]:https://www.influxdata.com/blog/influxdb-markedly-elasticsearch-in-time-series-data-metrics-benchmark/
[tick]:https://www.influxdata.com/time-series-platform/