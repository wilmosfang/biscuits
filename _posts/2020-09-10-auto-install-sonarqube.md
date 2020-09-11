---
layout: post
title: "Auto install sonarqube by compose"
date: 2020-09-10 02:37:42
image: '/assets/img/'
description: '自动安装 sonarqube'
main-class: 'code'
color: '#fbbd04' 
tags:
 - snippet
 - docker
 - tools
 - vagrant
 - sonarqube
 - devops
 - code
categories:
 - code
twitter_text: "Auto install sonarqube by compose"
introduction: "Auto install sonarqube by compose"
---

# 前言

开启一个 **CODE** 系列，用来收集各种好用的小片段

这一篇用来展示如何通过 docker-compose 来安装 sonarqube

> **Tip:** 展示中使用的各种软件版本可能不是最新，但我都会尽量提前交代

---

# 操作

## 依赖

* Docker 

~~~
[root@docker ~]# docker --version
Docker version 1.13.1, build 64e9980/1.13.1
[root@docker ~]#
~~~

## OS 环境

~~~
[root@docker ~]# hostnamectl
   Static hostname: docker
         Icon name: computer-vm
           Chassis: vm
        Machine ID: c7773881ade544df94677ad643d356d5
           Boot ID: 4c51606c39704637aeb61b9c8e4713b1
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@docker ~]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 82914sec preferred_lft 82914sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:7c:d7:94 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.210/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe7c:d794/64 scope link
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:12:18:a5:d1 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft fo
~~~

## 代码

~~~
[root@docker sonarqube_postgresql]# pwd
/docker/sonarqube_postgresql
[root@docker sonarqube_postgresql]# tree
.
├── docker-compose.yaml
└── init_sonarqube.bash

0 directories, 2 files
[root@docker sonarqube_postgresql]# cat init_sonarqube.bash
#!/bin/bash

## download docker compose
compose_version='1.27.0'
## sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/${compose_version}/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version

## kernal config
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096

## pull image
docker pull sonarqube:8.4.2-community
docker pull postgres:12.4

## service up

docker-compose -f docker-compose.yaml  up -d
[root@docker sonarqube_postgresql]# cat docker-compose.yaml
version: "3"

services:
  sonarqube:
    image: sonarqube:8.4.2-community
    ports:
      - "9000:9000"
    networks:
      - sonarnet
    environment:
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false"
      - sonar.jdbc.username=sonar
      - sonar.jdbc.password=sonar
    restart: always
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs

  db:
    image: postgres:12.4
    networks:
      - sonarnet
    environment:
      - POSTGRES_USER=sonar
      - POSTGRES_PASSWORD=sonar
    restart: always
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data

networks:
  sonarnet:
    driver: bridge

volumes:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:

[root@docker sonarqube_postgresql]#
~~~

## 执行

~~~
[root@docker sonarqube_postgresql]# time bash +x init_sonarqube.bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   423  100   423    0     0    486      0 --:--:-- --:--:-- --:--:--   485
100 11.6M  100 11.6M    0     0  4326k      0  0:00:02  0:00:02 --:--:-- 9169k
docker-compose version 1.27.0, build 980ec85b
vm.max_map_count = 262144
fs.file-max = 65536
Trying to pull repository docker.io/library/sonarqube ...
8.4.2-community: Pulling from docker.io/library/sonarqube
Digest: sha256:a0aac11dcf18e237af1de207402e222d1e251ebc21b2bbcdac2b9b87710f1dd3
Status: Image is up to date for docker.io/sonarqube:8.4.2-community
Trying to pull repository docker.io/library/postgres ...
12.4: Pulling from docker.io/library/postgres
d121f8d1c412: Pull complete
9f045f1653de: Pull complete
fa0c0f0a5534: Pull complete
54e26c2eb3f1: Pull complete
cede939c738e: Pull complete
69f99b2ba105: Pull complete
218ae2bec541: Pull complete
70a48a74e7cf: Pull complete
a5a0d51f9154: Pull complete
73e23a8be3f2: Pull complete
770413f2d2da: Pull complete
8ff552bf08c6: Pull complete
7f533d09024d: Pull complete
5a7614cf31d9: Pull complete
Digest: sha256:880dc88e23f925ee2b9be3c38d92cf00a99c251e8801258151340c281b5699e7
Status: Downloaded newer image for docker.io/postgres:12.4
Creating network "sonarqube_postgresql_sonarnet" with driver "bridge"
Creating volume "sonarqube_postgresql_sonarqube_conf" with default driver
Creating volume "sonarqube_postgresql_sonarqube_data" with default driver
Creating volume "sonarqube_postgresql_sonarqube_extensions" with default driver
Creating volume "sonarqube_postgresql_sonarqube_logs" with default driver
Creating volume "sonarqube_postgresql_postgresql" with default driver
Creating volume "sonarqube_postgresql_postgresql_data" with default driver
Creating sonarqube_postgresql_db_1        ... done
Creating sonarqube_postgresql_sonarqube_1 ... done

real	1m17.431s
user	0m0.957s
sys	0m0.404s
[root@docker sonarqube_postgresql]# docker-compose ps
              Name                            Command              State           Ports
-------------------------------------------------------------------------------------------------
sonarqube_postgresql_db_1          docker-entrypoint.sh postgres   Up      5432/tcp
sonarqube_postgresql_sonarqube_1   bin/run.sh bin/sonar.sh         Up      0.0.0.0:9000->9000/tcp
[root@docker sonarqube_postgresql]# docker ps -a
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                    NAMES
b7bc02fc5e4a        postgres:12.4               "docker-entrypoint..."   21 seconds ago      Up 19 seconds       5432/tcp                 sonarqube_postgresql_db_1
093a018c1567        sonarqube:8.4.2-community   "bin/run.sh bin/so..."   21 seconds ago      Up 19 seconds       0.0.0.0:9000->9000/tcp   sonarqube_postgresql_sonarqube_1
[root@docker sonarqube_postgresql]#
~~~


## 测试

访问

![sonarqube](/assets/img/sonarqube/sonarqube36.png)

登陆 admin/admin

![sonarqube](/assets/img/sonarqube/sonarqube37.png)

进入界面

![sonarqube](/assets/img/sonarqube/sonarqube38.png)


~~~
[root@jenkins tdd]# gradle sonarqube -Dsonar.host.url=http://10.0.0.210:9000/  -Dsonar.verbose=true

> Task :test

cn.xpbootcamp.gildedrose.AgedBrieTest > should_return_sellIn_1_quality_1_when_agedBrie_pass_1_day_given_sellIn_2_quality_0 PASSED

cn.xpbootcamp.gildedrose.AgedBrieTest > should_return_sellIn_n2_quality_22_when_agedBrie_pass_1_day_given_sellIn_n1_quality_20 PASSED

cn.xpbootcamp.gildedrose.AgedBrieTest > should_return_sellIn_1_quality_50_when_agedBrie_pass_1_day_given_sellIn_2_quality_49 PASSED

cn.xpbootcamp.gildedrose.AgedBrieTest > should_return_sellIn_1_quality_50_when_agedBrie_pass_1_day_given_sellIn_2_quality_50 PASSED

cn.xpbootcamp.gildedrose.AgedBrieTest > should_return_sellIn_n1_quality_22_when_agedBrie_pass_1_day_given_sellIn_0_quality_20 PASSED

cn.xpbootcamp.gildedrose.BackstagePassTest > should_return_sellIn_9_quality_47_when_backstagePass_pass_1_day_given_sellIn_10_quality_45 PASSED

cn.xpbootcamp.gildedrose.BackstagePassTest > should_return_sellIn_4_quality_48_when_backstagePass_pass_1_day_given_sellIn_5_quality_45 PASSED

cn.xpbootcamp.gildedrose.BackstagePassTest > should_return_sellIn_9_quality_50_when_backstagePass_pass_1_day_given_sellIn_10_quality_49 PASSED

cn.xpbootcamp.gildedrose.BackstagePassTest > should_return_sellIn_9_quality_50_when_backstagePass_pass_1_day_given_sellIn_10_quality_50 PASSED

cn.xpbootcamp.gildedrose.BackstagePassTest > should_return_sellIn_n1_quality_0_when_backstagePass_pass_1_day_given_sellIn_0_quality_20 PASSED

cn.xpbootcamp.gildedrose.BackstagePassTest > should_return_sellIn_4_quality_50_when_backstagePass_pass_1_day_given_sellIn_5_quality_49 PASSED

cn.xpbootcamp.gildedrose.BackstagePassTest > should_return_sellIn_8_quality_47_when_backstagePass_pass_1_day_given_sellIn_9_quality_45 PASSED

cn.xpbootcamp.gildedrose.BackstagePassTest > should_return_sellIn_0_quality_23_when_backstagePass_pass_1_day_given_sellIn_1_quality_20 PASSED

cn.xpbootcamp.gildedrose.BackstagePassTest > should_return_sellIn_14_quality_21_when_backstagePass_pass_1_day_given_sellIn_15_quality_20 PASSED

cn.xpbootcamp.gildedrose.NormalGoodsTest > should_return_sellIn_1_quality_0_when_normal_product_pass_1_day_given_sellIn_2_quality_0 PASSED

cn.xpbootcamp.gildedrose.NormalGoodsTest > should_return_sellIn_n2_quality_4_when_normal_product_pass_1_day_given_sellIn_n1_quality_6 PASSED

cn.xpbootcamp.gildedrose.NormalGoodsTest > should_return_sellIn_2_quality_5_when_normal_product_pass_1_day_given_sellIn_3_quality_6 PASSED

cn.xpbootcamp.gildedrose.NormalGoodsTest > should_return_sellIn_9_quality_19_when_normal_product_pass_1_day_given_sellIn_10_quality_20 PASSED

cn.xpbootcamp.gildedrose.NormalGoodsTest > should_return_sellIn_n1_quality_4_when_normal_product_pass_1_day_given_sellIn_0_quality_6 PASSED

cn.xpbootcamp.gildedrose.NormalGoodsTest > should_return_sellIn_2_quality_50_when_normal_product_pass_1_day_given_sellIn_3_quality_51 PASSED

cn.xpbootcamp.gildedrose.SulfurasTest > should_return_sellIn_n1_quality_1_when_sulfuras_pass_1_day_given_sellIn_n1_quality_1 PASSED

cn.xpbootcamp.gildedrose.SulfurasTest > should_return_sellIn_n1_quality_45_when_sulfuras_pass_1_day_given_sellIn_n1_quality_45 PASSED

cn.xpbootcamp.gildedrose.SulfurasTest > should_return_sellIn_n2_quality_1_when_sulfuras_pass_1_day_given_sellIn_n2_quality_1 PASSED

cn.xpbootcamp.gildedrose.SulfurasTest > should_return_sellIn_n1_quality_50_when_sulfuras_pass_1_day_given_sellIn_n1_quality_50 PASSED

cn.xpbootcamp.gildedrose.SulfurasTest > should_return_sellIn_0_quality_45_when_sulfuras_pass_1_day_given_sellIn_0_quality_45 PASSED

-----------------------------------------------------------------
 Result: SUCCESS (25 Tests, 25 Successes, 0 Failures, 0 Skipped)
-----------------------------------------------------------------

> Task :sonarqube
SonarScanner will require Java 11 to run starting in SonarQube 8.x

BUILD SUCCESSFUL in 39s
4 actionable tasks: 4 executed
[root@jenkins tdd]#
~~~

可以看到分析结果

![sonarqube](/assets/img/sonarqube/sonarqube39.png)



---

# 总结

通过 docker-compose 可以快速组建服务

是一种轻量的的编排工具，非常好用

* TOC
{:toc}

---
