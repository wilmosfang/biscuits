---
layout: post
title: "Auto install ATC by compose"
date: 2020-09-11 04:37:42
image: '/assets/img/'
description: '自动安装敏捷工具链'
main-class: 'code'
color: '#fbbd04' 
tags:
 - snippet
 - docker
 - tools
 - postgresql
 - jenkins
 - devops
 - sonarqube
 - gitea
 - code
categories:
 - code
twitter_text: "Auto install ATC by compose"
introduction: "Auto install ATC by compose"
---

# 前言

开启一个 **CODE** 系列，用来收集各种好用的小片段

这一篇用来展示如何通过 docker-compose 构建一个最简单的敏捷工具链

里面包含数据库、代码仓库、流水线、和质量评估系统

是一个极简的 CI 工具链

分别由 **postgresql、gitea、jenkins、sonarqube** 来构成

> **Tip:** 此次使用的镜像版本为 **gitea:1.12.4、jenkins:2.60.3、sonarqube:8.4.2-community、postgres:12.4**

---

# 操作

## 依赖

* Docker 

~~~
[root@tc atc]# docker --version
Docker version 1.13.1, build 64e9980/1.13.1
[root@tc atc]#
~~~

## OS 环境

~~~
[root@tc atc]# hostnamectl
   Static hostname: tc
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 48338289faac42af94fb43464ec98934
           Boot ID: 8884d94904e44a4b834ca316d8d201a9
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@tc atc]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 85716sec preferred_lft 85716sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:58:48:35 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.223/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe58:4835/64 scope link
       valid_lft forever preferred_lft forever
4: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:8e:bf:d2:8e brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
[root@tc atc]#
~~~

## 代码

~~~
[root@tc atc]# pwd
/atc/atc
[root@tc atc]# tree
.
├── app.ini
├── docker-compose.yaml
├── hudson.model.UpdateCenter.xml
├── init-atc.bash
├── init-sonar-db.sh
├── jenkins.war.2.249.1
└── jenkins.war.2.89.3

0 directories, 7 files
[root@tc atc]# cat init-atc.bash
#!/bin/bash

## ensure compose
if [ ! -e /usr/local/bin/docker-compose ]; then
  ## download docker compose
  compose_version='1.27.0'
  ## sudo curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
  sudo curl -L https://get.daocloud.io/docker/compose/releases/download/${compose_version}/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
  sudo chmod +x /usr/local/bin/docker-compose
  docker-compose --version
fi

## kernal config
sysctl -w vm.max_map_count=262144
sysctl -w fs.file-max=65536
ulimit -n 65536
ulimit -u 4096

## pull image
docker pull gitea/gitea:1.12.4
docker pull jenkins:2.60.3
docker pull sonarqube:8.4.2-community
#docker pull sonarqube:6.7.1
docker pull postgres:12.4

## service up
docker-compose -f docker-compose.yaml  up -d
[root@tc atc]# cat docker-compose.yaml
version: "3"

networks:
  atcnet:
    driver: bridge

volumes:
  sonarqube_conf:
  sonarqube_data:
  sonarqube_extensions:
  sonarqube_logs:
  postgresql:
  postgresql_data:
  gitea_data:
  jenkins_home:

services:
  db:
    image: postgres:12.4
    networks:
      - atcnet
    environment:
      - POSTGRES_USER=pguser
      - POSTGRES_PASSWORD=pgpass
      - POSTGRES_DB=gitea
    restart: always
    volumes:
      - postgresql:/var/lib/postgresql
      - postgresql_data:/var/lib/postgresql/data
      - ./init-sonar-db.sh:/docker-entrypoint-initdb.d/init-sonar-db.sh

  gitea:
    image: gitea/gitea:1.8.3
    environment:
      - USER_UID=1000
      - USER_GID=1000
      - DB_TYPE=postgres
      - DB_HOST=db:5432
      - DB_NAME=gitea
      - DB_USER=pguser
      - DB_PASSWD=pgpass
      - SSH_DOMAIN=10.0.0.223
      - DOMAIN=10.0.0.223
      - ROOT_URL=http://10.0.0.223:3000/
    ports:
      - "3000:3000"
      - "10022:22"
    depends_on:
      - db
    networks:
      - atcnet
    restart: always
    volumes:
      - gitea_data:/data
      - /etc/timezone:/etc/timezone:ro
      - /etc/localtime:/etc/localtime:ro
      - ./app.ini:/data/gitea/conf/app.ini

  sonarqube:
    image: sonarqube:8.4.2-community
#    image: sonarqube:6.7.1
    ports:
      - "9000:9000"
    networks:
      - atcnet
    environment:
      # - sonar.jdbc.url="jdbc:postgresql://db:5432/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false"
      - sonar.jdbc.url=jdbc:postgresql://db:5432/sonar
      - sonar.jdbc.username=pguser
      - sonar.jdbc.password=pgpass
    restart: always
    volumes:
      - sonarqube_conf:/opt/sonarqube/conf
      - sonarqube_data:/opt/sonarqube/data
      - sonarqube_extensions:/opt/sonarqube/extensions
      - sonarqube_logs:/opt/sonarqube/logs

  jenkins:
    image: jenkins:2.60.3
    #image: jenkins/jenkins:lts
    ports:
      - "8080:8080"
      - "50000:50000"
    networks:
      - atcnet
    restart: always
    volumes:
      - jenkins_home:/var/jenkins_home
      - ./hudson.model.UpdateCenter.xml:/var/jenkins_home/hudson.model.UpdateCenter.xml
      - ./jenkins.war.2.249.1:/usr/share/jenkins/jenkins.war  #upgrade to jenkins.war.2.249.1
[root@tc atc]# cat init-sonar-db.sh
#!/bin/bash
set -e

psql -v ON_ERROR_STOP=1 --username "$POSTGRES_USER" --dbname "$POSTGRES_DB" <<-EOSQL
    CREATE DATABASE sonar;
    GRANT ALL PRIVILEGES ON DATABASE sonar TO "$POSTGRES_USER";
EOSQL
[root@tc atc]# cat app.ini
APP_NAME = Gitea: Git with a cup of tea
RUN_MODE = prod
RUN_USER = git

[repository]
ROOT = /data/git/repositories

[repository.local]
LOCAL_COPY_PATH = /data/gitea/tmp/local-repo

[repository.upload]
TEMP_PATH = /data/gitea/uploads

[server]
APP_DATA_PATH    = /data/gitea
SSH_DOMAIN       = 10.0.0.223
HTTP_PORT        = 3000
ROOT_URL         = http://10.0.0.223:3000/
DISABLE_SSH      = false
SSH_PORT         = 22
LFS_CONTENT_PATH = /data/git/lfs
DOMAIN           = 10.0.0.223
LFS_START_SERVER = true
LFS_JWT_SECRET   = Cd6julUMHUa-ZckMu9HqnLWPEZ9NjlqwrD997fwO0TA
OFFLINE_MODE     = false

[database]
PATH     = /data/gitea/gitea.db
DB_TYPE  = postgres
HOST     = db:5432
NAME     = gitea
USER     = pguser
PASSWD   = pgpass
SSL_MODE = disable

[indexer]
ISSUE_INDEXER_PATH = /data/gitea/indexers/issues.bleve

[session]
PROVIDER_CONFIG = /data/gitea/sessions
PROVIDER        = file

[picture]
AVATAR_UPLOAD_PATH      = /data/gitea/avatars
DISABLE_GRAVATAR        = false
ENABLE_FEDERATED_AVATAR = true

[attachment]
PATH = /data/gitea/attachments

[log]
ROOT_PATH = /data/gitea/log
MODE      = file
LEVEL     = Info

[security]
INSTALL_LOCK   = true
SECRET_KEY     = ZMa1vvIz85K0LBc8RgTmNrYyllxt9xa66iThZt4JdcSKzld0Mdk2frxwc6By2VuH
INTERNAL_TOKEN = eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYmYiOjE1OTk4MTMxNDB9.9h8r5DSvG2i23lDgoKbRRB2YnMRuGGayssJd_SwlfCg

[service]
DISABLE_REGISTRATION              = false
REQUIRE_SIGNIN_VIEW               = false
REGISTER_EMAIL_CONFIRM            = false
ENABLE_NOTIFY_MAIL                = false
ALLOW_ONLY_EXTERNAL_REGISTRATION  = false
ENABLE_CAPTCHA                    = false
DEFAULT_KEEP_EMAIL_PRIVATE        = false
DEFAULT_ALLOW_CREATE_ORGANIZATION = true
DEFAULT_ENABLE_TIMETRACKING       = true
NO_REPLY_ADDRESS                  = noreply.example.org

[oauth2]
JWT_SECRET = 1QhUrKCnq1P7u8sPn3HmnTAyxYb_FktcERlxmnE2nIE

[mailer]
ENABLED = false

[openid]
ENABLE_OPENID_SIGNIN = true
ENABLE_OPENID_SIGNUP = true

[root@tc atc]# cat hudson.model.UpdateCenter.xml
<?xml version='1.1' encoding='UTF-8'?>
<sites>
  <site>
    <id>default</id>
    <url>https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json</url>
  </site>
</sites>[root@tc atc]#
[root@tc atc]#
[root@tc atc]#
[root@tc atc]#
[root@tc atc]# tree
.
├── app.ini
├── docker-compose.yaml
├── hudson.model.UpdateCenter.xml
├── init-atc.bash
├── init-sonar-db.sh
├── jenkins.war.2.249.1
└── jenkins.war.2.89.3

0 directories, 7 files
[root@tc atc]#
~~~


> **Note:** 没有更新 **hudson.model.UpdateCenter.xml** 的情况下，容易在启动时，卡在 **Please wait while Jenkins is getting ready to work ...** 这一步，原因是访问 **https://updates.jenkins.io/update-center.json** 受阻，无法正常执行如下几步

~~~
2020-09-16 16:40:23.212+0000 [id=41]	INFO	h.m.DownloadService$Downloadable#load: Obtained the updated data file for hudson.tasks.Maven.MavenInstaller
2020-09-16 16:40:23.213+0000 [id=41]	INFO	hudson.util.Retrier#start: Performed the action check updates server successfully at the attempt #1
2020-09-16 16:40:23.215+0000 [id=41]	INFO	hudson.model.AsyncPeriodicWork#lambda$doRun$0: Finished Download metadata. 6,791 ms
2020-09-16 16:40:23.240+0000 [id=26]	INFO	jenkins.InitReactorRunner$1#onAttained: Completed initialization
2020-09-16 16:40:23.254+0000 [id=19]	INFO	hudson.WebAppMain$3#run: Jenkins is fully up and running
~~~

解决办法是就替换这个URL地址到一个容易访问的更新中心，我们这里更新成了 **https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json**

## 执行

~~~
[root@tc atc]# time bash +x init-atc.bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   423  100   423    0     0    570      0 --:--:-- --:--:-- --:--:--   570
100 11.6M  100 11.6M    0     0  3626k      0  0:00:03  0:00:03 --:--:-- 8533k
docker-compose version 1.27.0, build 980ec85b
vm.max_map_count = 262144
fs.file-max = 65536
Trying to pull repository docker.io/gitea/gitea ...
1.12.4: Pulling from docker.io/gitea/gitea
cbdbe7a5bc2a: Pull complete
441146be90c1: Pull complete
d1f126a39d17: Pull complete
8128cb1db906: Pull complete
974dc3b79b9b: Downloading [===>                                               ] 2.726 MB/44.47 MB
b03db1e8281f: Download complete


^C
real	10m25.002s
user	0m0.480s
sys	0m0.275s

[root@tc atc]# time bash +x init-atc.bash
vm.max_map_count = 262144
fs.file-max = 65536
Trying to pull repository docker.io/gitea/gitea ...
1.12.4: Pulling from docker.io/gitea/gitea
cbdbe7a5bc2a: Pull complete
441146be90c1: Pull complete
d1f126a39d17: Pull complete
8128cb1db906: Pull complete
974dc3b79b9b: Pull complete
b03db1e8281f: Pull complete
Digest: sha256:85416d6f65fecc66d156fcbf0789175cd1b7e1b37094733a8caaf12489e091ba
Status: Downloaded newer image for docker.io/gitea/gitea:1.12.4
Trying to pull repository docker.io/library/jenkins ...
2.60.3: Pulling from docker.io/library/jenkins
55cbf04beb70: Pull complete
1607093a898c: Pull complete
9a8ea045c926: Pull complete
d4eee24d4dac: Pull complete
c58988e753d7: Pull complete
794a04897db9: Pull complete
70fcfa476f73: Pull complete
0539c80a02be: Pull complete
54fefc6dcf80: Pull complete
911bc90e47a8: Pull complete
38430d93efed: Pull complete
7e46ccda148a: Pull complete
c0cbcb5ac747: Pull complete
35ade7a86a8e: Pull complete
aa433a6a56b1: Pull complete
841c1dd38d62: Pull complete
b865dcb08714: Pull complete
5a3779030005: Pull complete
12b47c68955c: Pull complete
1322ea3e7bfd: Pull complete
Digest: sha256:eeb4850eb65f2d92500e421b430ed1ec58a7ac909e91f518926e02473904f668
Status: Downloaded newer image for docker.io/jenkins:2.60.3
Trying to pull repository docker.io/library/sonarqube ...
8.4.2-community: Pulling from docker.io/library/sonarqube
cbdbe7a5bc2a: Already exists
02126857e210: Pull complete
b35a2496a4dd: Pull complete
8847622e995e: Pull complete
8d61c5b43a5f: Pull complete
Digest: sha256:a0aac11dcf18e237af1de207402e222d1e251ebc21b2bbcdac2b9b87710f1dd3
Status: Downloaded newer image for docker.io/sonarqube:8.4.2-community
Trying to pull repository docker.io/library/sonarqube ...
6.7.1: Pulling from docker.io/library/sonarqube
c73ab1c6897b: Pull complete
1ab373b3deae: Pull complete
b542772b4177: Pull complete
57c8de432dbe: Pull complete
da44f64ae999: Pull complete
0bbc7b377a91: Pull complete
1b6c70b3786f: Pull complete
48010c1717c7: Pull complete
7a6123cacadf: Pull complete
1c41217c08a9: Pull complete
fbcc726b62ba: Pull complete
5a07939089bd: Pull complete
5e233fc91ef7: Pull complete
Digest: sha256:c2503982297455c67d2dec8115b19534d2fd98db1951d098f4fcbb885aaabb74
Status: Downloaded newer image for docker.io/sonarqube:6.7.1
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
Digest: sha256:130c7981159ecaead8857e576a0aabbaffbcc61647cf18f3240cf804d8f68cab
Status: Downloaded newer image for docker.io/postgres:12.4
Creating network "atc_atcnet" with driver "bridge"
Creating volume "atc_sonarqube_conf" with default driver
Creating volume "atc_sonarqube_data" with default driver
Creating volume "atc_sonarqube_extensions" with default driver
Creating volume "atc_sonarqube_logs" with default driver
Creating volume "atc_postgresql" with default driver
Creating volume "atc_postgresql_data" with default driver
Creating volume "atc_gitea_data" with default driver
Creating volume "atc_jenkins_home" with default driver
Pulling gitea (gitea/gitea:1.8.3)...
Trying to pull repository docker.io/gitea/gitea ...
1.8.3: Pulling from docker.io/gitea/gitea
e7c96db7181b: Pull complete
495d5d982a9a: Pull complete
5fe15899a512: Pull complete
3448674ddbcd: Pull complete
c5105adf5178: Pull complete
2d30e4c01df3: Pull complete
Digest: sha256:25af63fb9625051c220c2e1ecb7315dad0c6013db7b4dcc2830d046a0b5c7204
Status: Downloaded newer image for docker.io/gitea/gitea:1.8.3
Creating atc_jenkins_1   ... done
Creating atc_db_1        ... done
Creating atc_sonarqube_1 ... done
Creating atc_gitea_1     ... done

real	6m55.006s
user	0m0.956s
sys	0m0.458s
[root@tc atc]#
[root@tc atc]# docker-compose ps
     Name                    Command               State                        Ports
-----------------------------------------------------------------------------------------------------------
atc_db_1          docker-entrypoint.sh postgres    Up      5432/tcp
atc_gitea_1       /usr/bin/entrypoint /bin/s ...   Up      0.0.0.0:10022->22/tcp, 0.0.0.0:3000->3000/tcp
atc_jenkins_1     /bin/tini -- /usr/local/bi ...   Up      0.0.0.0:50000->50000/tcp, 0.0.0.0:8080->8080/tcp
atc_sonarqube_1   bin/run.sh bin/sonar.sh          Up      0.0.0.0:9000->9000/tcp
[root@tc atc]# docker ps -a
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                              NAMES
123bb39154ca        gitea/gitea:1.8.3           "/usr/bin/entrypoi..."   2 minutes ago       Up 2 minutes        0.0.0.0:3000->3000/tcp, 0.0.0.0:10022->22/tcp      atc_gitea_1
e378fb7e8db7        postgres:12.4               "docker-entrypoint..."   2 minutes ago       Up 2 minutes        5432/tcp                                           atc_db_1
7fc44572e532        sonarqube:8.4.2-community   "bin/run.sh bin/so..."   2 minutes ago       Up 2 minutes        0.0.0.0:9000->9000/tcp                             atc_sonarqube_1
2b29f49c502d        jenkins:2.60.3              "/bin/tini -- /usr..."   2 minutes ago       Up 2 minutes        0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   atc_jenkins_1
[root@tc atc]#
~~~

> **Tip:** 中断重试有时可以加速下载，因为挑选了其它CDN节点




## 访问与基础配置

sonarqube 的访问界面

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc01.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc02.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc03.png)

gitea 的访问界面

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc04.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc05.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc06.png)

jenkins 的访问界面

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc07.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc08.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc09.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc10.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc11.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc12.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc13.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc14.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc15.png)

![atc](/assets/img/post/2020-09-16-auto-install-atc/atc16.png)


postgresql 的访问

~~~
[root@tc atc]# docker ps -a
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                                              NAMES
b383468b5b40        jenkins:2.60.3              "/bin/tini -- /usr..."   24 minutes ago      Up 24 minutes       0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   atc_jenkins_1
123bb39154ca        gitea/gitea:1.8.3           "/usr/bin/entrypoi..."   2 hours ago         Up 40 minutes       0.0.0.0:3000->3000/tcp, 0.0.0.0:10022->22/tcp      atc_gitea_1
e378fb7e8db7        postgres:12.4               "docker-entrypoint..."   2 hours ago         Up 40 minutes       5432/tcp                                           atc_db_1
7fc44572e532        sonarqube:8.4.2-community   "bin/run.sh bin/so..."   2 hours ago         Up 40 minutes       0.0.0.0:9000->9000/tcp                             atc_sonarqube_1
[root@tc atc]# docker exec -it e378fb7e8db7 psql -U pguser -h db -d gitea
Password for user pguser:
psql (12.4 (Debian 12.4-1.pgdg100+1))
Type "help" for help.

gitea=# \l
                              List of databases
   Name    | Owner  | Encoding |  Collate   |   Ctype    | Access privileges
-----------+--------+----------+------------+------------+-------------------
 gitea     | pguser | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres  | pguser | UTF8     | en_US.utf8 | en_US.utf8 |
 sonar     | pguser | UTF8     | en_US.utf8 | en_US.utf8 | =Tc/pguser       +
           |        |          |            |            | pguser=CTc/pguser
 template0 | pguser | UTF8     | en_US.utf8 | en_US.utf8 | =c/pguser        +
           |        |          |            |            | pguser=CTc/pguser
 template1 | pguser | UTF8     | en_US.utf8 | en_US.utf8 | =c/pguser        +
           |        |          |            |            | pguser=CTc/pguser
(5 rows)

gitea=# exit
[root@tc atc]#
~~~


---

# 总结

通过 docker-compose 可以快速组建服务

是一种轻量的的编排工具，非常好用

后面以一个代码库为示例来演示如何组织这几个软件对外提供服务

* TOC
{:toc}

---
