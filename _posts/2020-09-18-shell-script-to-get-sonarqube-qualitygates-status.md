---
layout: post
title: "Get Sonarqube Gualitygate status by Shell"
date: 2020-09-18 01:37:42
image: '/assets/img/'
description: '通过脚本自动获取sonarqube质量门禁结果'
main-class: 'code'
color: '#fbbd04' 
tags:
 - snippet
 - bash
 - tools
 - jenkins
 - devops
 - sonarqube
 - code
 - jq
categories:
 - code
twitter_text: "Get Sonarqube Gualitygate status by Shell"
introduction: "Get Sonarqube Gualitygate status by Shell"
---

# 前言

开启一个 **CODE** 系列，用来收集各种好用的小片段

这一篇用来展示如何通过 shell 脚本来自动获取 sonarqube 质量门禁的结果

> **Tip:** 此次使用的镜像版本为 **jenkins:2.60.3、sonarqube:8.4.2-community、jq-1.6**

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
[root@atc atc]# hostnamectl
   Static hostname: atc
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 68aa1110196d499eabb91ec895f78152
           Boot ID: 8d6e311f38ee4a8abd89ea280a7972f1
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
[root@atc atc]# ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 82668sec preferred_lft 82668sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:cc:44:e0 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.222/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fecc:44e0/64 scope link
       valid_lft forever preferred_lft forever
4: br-18d0fb840bb4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:13:2c:d1:75 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.1/16 scope global br-18d0fb840bb4
       valid_lft forever preferred_lft forever
    inet6 fe80::42:13ff:fe2c:d175/64 scope link
       valid_lft forever preferred_lft forever
5: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default
    link/ether 02:42:bd:11:87:f3 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 scope global docker0
       valid_lft forever preferred_lft forever
43: veth871a365@if42: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-18d0fb840bb4 state UP group default
    link/ether 1a:39:e3:36:cc:70 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::1839:e3ff:fe36:cc70/64 scope link
       valid_lft forever preferred_lft forever
45: vethbd44953@if44: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-18d0fb840bb4 state UP group default
    link/ether aa:d9:6e:39:1a:0d brd ff:ff:ff:ff:ff:ff link-netnsid 2
    inet6 fe80::a8d9:6eff:fe39:1a0d/64 scope link
       valid_lft forever preferred_lft forever
47: veth9cd0f11@if46: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-18d0fb840bb4 state UP group default
    link/ether 12:99:f2:e2:a7:1b brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::1099:f2ff:fee2:a71b/64 scope link
       valid_lft forever preferred_lft forever
49: vethe2650b8@if48: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master br-18d0fb840bb4 state UP group default
    link/ether 02:e7:ed:7a:06:d5 brd ff:ff:ff:ff:ff:ff link-netnsid 3
    inet6 fe80::e7:edff:fe7a:6d5/64 scope link
       valid_lft forever preferred_lft forever
[root@atc atc]#
~~~

## 代码

~~~
[root@atc atc]# ls -l check_qg.bash
-rwxr-xr-x. 1 502 games 1755 Sep 17 18:15 check_qg.bash
[root@atc atc]#
[root@atc atc]# cat check_qg.bash
#!/bin/bash


help(){
cat  <<EOF
"Usage: `basename $0` [options] "
    -c <COUNT>: num to retry, default 3 times
    -i <INTERVAL>: interval to retry, default 10 seconds
    -p <REPORT_PATH>: path of report-task.txt, default is ".scannerwork/report-task.txt"
EOF
exit 1
}

while getopts 'c:i:p:' OPT; do
    case $OPT in
        c)
            COUNT=${OPTARG:-3};;
        i)
            INTERVAL=${OPTARG:-10};;
        p)
            REPORT_PATH=${OPTARG:-".scannerwork/report-task.txt"};;
        *)
          help
    esac
done

COUNT=${COUNT:-3}
INTERVAL=${INTERVAL:-10}
REPORT_PATH=${REPORT_PATH:-".scannerwork/report-task.txt"}

if [ $# == 0 ]
then
  help
fi

if [ ! -f ${REPORT_PATH} ]
then
  echo "${REPORT_PATH} not exist"
  help
fi

#echo $#
#echo ${COUNT}
#echo ${INTERVAL}
#echo ${REPORT_PATH}

task_status_url=`grep ceTaskUrl ${REPORT_PATH} | cut -f 2- -d'='`
#task_status_url='http://10.0.0.222:9000/api/ce/task?id=AXSbT_Jclc5tYZIC4w-u'
server_url=`grep serverUrl ${REPORT_PATH} | cut -f 2- -d'='`
#server_url='http://10.0.0.222:9000'
project_status_api='api/qualitygates/project_status'

for ((i=1;i<=${COUNT};i++))
do
   task_status=`curl ${task_status_url} 2>/dev/null | jq .task.status`
   if [ "${task_status}"x = '"SUCCESS"'x ];
   then
      break
   else
     sleep ${INTERVAL}
     echo "retry ${i}"
   fi
done

analysisId=`curl ${task_status_url} 2>/dev/null | jq .task.analysisId | sed 's/"//g'`
project_status_url=${server_url}'/'${project_status_api}'?analysisId='${analysisId}

project_status=`curl ${project_status_url} 2>/dev/null | jq .projectStatus.status`

if [ "${project_status}"x == '"OK"'x ];
then
  echo "project_status is: ${project_status}"
  exit 0
else
  echo "project_status is: ${project_status}"
  exit 1
fi

[root@atc atc]#
~~~



## 执行

~~~
[root@atc atc]# docker ps -a  | grep jenkins
a6e40806d7da        jenkins:2.60.3              "/bin/tini -- /usr..."   52 minutes ago      Up 51 minutes       0.0.0.0:8080->8080/tcp, 0.0.0.0:50000->50000/tcp   atc_jenkins_1
[root@atc atc]# docker cp ./check_qg.bash   a6e40806d7da:/tmp/
[root@atc atc]# docker exec -it a6e40806d7da /bin/bash
jenkins@a6e40806d7da:/$ cd /var/jenkins_home/workspace/tdd
jenkins@a6e40806d7da:~/workspace/tdd$ /tmp/check_qg.bash
"Usage: check_qg.bash [options] "
    -c <COUNT>: num to retry, default 3 times
    -i <INTERVAL>: interval to retry, default 10 seconds
    -p <REPORT_PATH>: path of report-task.txt, default is ".scannerwork/report-task.txt"
jenkins@a6e40806d7da:~/workspace/tdd$ /tmp/check_qg.bash  -c 4
project_status is: "OK"
jenkins@a6e40806d7da:~/workspace/tdd$ bash -x /tmp/check_qg.bash -c 4
+ getopts c:i:p: OPT
+ case $OPT in
+ COUNT=4
+ getopts c:i:p: OPT
+ COUNT=4
+ INTERVAL=10
+ REPORT_PATH=.scannerwork/report-task.txt
+ '[' 2 == 0 ']'
+ '[' '!' -f .scannerwork/report-task.txt ']'
++ grep ceTaskUrl .scannerwork/report-task.txt
++ cut -f 2- -d=
+ task_status_url='http://10.0.0.222:9000/api/ce/task?id=AXSbT_Jclc5tYZIC4w-u'
++ grep serverUrl .scannerwork/report-task.txt
++ cut -f 2- -d=
+ server_url=http://10.0.0.222:9000
+ project_status_api=api/qualitygates/project_status
+ (( i=1 ))
+ (( i<=4 ))
++ jq .task.status
++ curl 'http://10.0.0.222:9000/api/ce/task?id=AXSbT_Jclc5tYZIC4w-u'
+ task_status='"SUCCESS"'
+ '[' '"SUCCESS"x' = '"SUCCESS"x' ']'
+ break
++ curl 'http://10.0.0.222:9000/api/ce/task?id=AXSbT_Jclc5tYZIC4w-u'
++ jq .task.analysisId
++ sed 's/"//g'
+ analysisId=AXSbT_TJJ0ld7-ai8wKB
+ project_status_url='http://10.0.0.222:9000/api/qualitygates/project_status?analysisId=AXSbT_TJJ0ld7-ai8wKB'
++ curl 'http://10.0.0.222:9000/api/qualitygates/project_status?analysisId=AXSbT_TJJ0ld7-ai8wKB'
++ jq .projectStatus.status
+ project_status='"OK"'
+ '[' '"OK"x' == '"OK"x' ']'
+ echo 'project_status is: "OK"'
project_status is: "OK"
+ exit 0
jenkins@a6e40806d7da:~/workspace/tdd$
~~~

> **Tip:** 可以定制重试次数 **`-c`**, 重试间隔 **`-i`**, 指定报告路径 **`-p`**
 
参考的逻辑如下：

**https://docs.sonarqube.org/display/SONARQUBE53/Breaking+the+CI+Build**

1. call the analysis's ceTaskUrl and examine the "status" value:

~~~
a. PENDING or IN_PROGRESS - check again later
b. FAILED or CANCELED - break the build?
c. SUCCESS - move forward
~~~

2. call Quality Gate web service for status


* Workflow of WS API Calls

~~~
1. Search in <work_dir>/report-task.txt the values of the CE Task URL (ceTaskUrl) and CE Task Id (ceTaskId)
2. Call /api/ce/task?id=XXX where XXX is the CE Task Id retrieved from <work_dir>/report-task.txt (save it as analysisId somewhere)
3. Do the Step 2 until the status is SUCCESS, CANCELED or FAILED
4. If the status is FAILED => break the build
5. If the status is SUCCESS
    a. Use the analysisId from the JSON returned by /api/ce/task?id=XXX
    b. Immediately call /api/qualitygates/project_status?analysisId=YYY to check the status of the quality gate
~~~

参考的 API 文档为

**https://next.sonarqube.com/sonarqube/web_api/api/qualitygates**

>**Tip:** 使用 **[jq][jq]** 来解析 json 会轻松很多

---

# 总结

通过 shell 来直接调用 API 获取门禁结果，

* TOC
{:toc}

---

[jq]:https://stedolan.github.io/jq/download/