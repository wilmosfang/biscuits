---
layout: post
title: "SonarScanner for Gradle"
date: 2020-01-20 00:46:36
image: '/assets/img/'
description: 'SonarScanner 集成到 Gradle'
main-class: tools
color: 
tags: 
 - gradle
 - tools
 - sonarqube
 - java
categories: 
 - sonarqube
twitter_text: 'sonarscanner for gradle'
introduction: 'sonarscanner for gradle'
---


# 前言

**[SonarQube][sonarqube]** 是一个很受欢迎的静态代码扫描软件

>Your teammate for Code Quality and Security
>SonarQube empowers all developers to write cleaner and safer code

能非常方便地集成到 CI/CD 环境中

这里分享一下 **[SonarQube][sonarqube]** 与 Gradle 的集成方法

参考 **[SonarScanner for Gradle][sonarscanner_for_gradle]**

> **Tip:** 当前版本 **SonarQube Version: 8.1** Release on December 2019

---

# 操作

## 系统环境

~~~
(base) [vagrant@workspace demo]$ hostnamectl
   Static hostname: workspace
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 583200922eeb40808cf8c6e488ba09c4
           Boot ID: 062dcff24e50470da8159fb3cb9cb8f2
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-957.12.2.el7.x86_64
      Architecture: x86-64
(base) [vagrant@workspace demo]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 81226sec preferred_lft 81226sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:3a:8f:13 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.100/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe3a:8f13/64 scope link
       valid_lft forever preferred_lft forever
(base) [vagrant@workspace demo]$ gradle --version

------------------------------------------------------------
Gradle 6.1
------------------------------------------------------------

Build time:   2020-01-15 23:56:46 UTC
Revision:     539d277fdba571ebcc9617a34329c83d7d2b259e

Kotlin:       1.3.61
Groovy:       2.5.8
Ant:          Apache Ant(TM) version 1.10.7 compiled on September 1 2019
JVM:          1.8.0_232 (Oracle Corporation 25.232-b09)
OS:           Linux 3.10.0-957.12.2.el7.x86_64 amd64

(base) [vagrant@workspace demo]$
~~~


## 软件版本

java 版本为 **1.8.0_232**

~~~
(base) [vagrant@workspace demo]$ java -version
openjdk version "1.8.0_232"
OpenJDK Runtime Environment (build 1.8.0_232-b09)
OpenJDK 64-Bit Server VM (build 25.232-b09, mixed mode)
(base) [vagrant@workspace demo]$
~~~

Gradle 版本为 **Gradle 6.1**

~~~
(base) [vagrant@workspace demo]$ gradle --version

------------------------------------------------------------
Gradle 6.1
------------------------------------------------------------

Build time:   2020-01-15 23:56:46 UTC
Revision:     539d277fdba571ebcc9617a34329c83d7d2b259e

Kotlin:       1.3.61
Groovy:       2.5.8
Ant:          Apache Ant(TM) version 1.10.7 compiled on September 1 2019
JVM:          1.8.0_232 (Oracle Corporation 25.232-b09)
OS:           Linux 3.10.0-957.12.2.el7.x86_64 amd64

(base) [vagrant@workspace demo]$
~~~

SonarQube 的版本为 **Community EditionVersion 8.1 (build 31237)**

## SonarQube 的服务地址

服务地址为 **`http://10.0.0.13:8000/sonarqube`**

~~~
[sonar@sonarqube conf]$ pwd
/home/sonar/sonarqube-8.1.0.31237/conf
[sonar@sonarqube conf]$ grep -i 'sonar.web' sonar.properties  | grep -v "#"
sonar.web.host=0.0.0.0
sonar.web.context=/sonarqube
sonar.web.port=8000
[sonar@sonarqube conf]$ ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 52:54:00:8a:fe:e6 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global noprefixroute dynamic eth0
       valid_lft 80531sec preferred_lft 80531sec
    inet6 fe80::5054:ff:fe8a:fee6/64 scope link 
       valid_lft forever preferred_lft forever
3: eth1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:6b:b4:f8 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.13/24 brd 10.0.0.255 scope global noprefixroute eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe6b:b4f8/64 scope link 
       valid_lft forever preferred_lft forever
[sonar@sonarqube conf]$ 
~~~


## 配置 Gradle 加入 plugin

在 plugins 里面加入 **`id "org.sonarqube" version "2.8"`**

~~~
(base) [vagrant@workspace demo]$ cat build.gradle
plugins {
	id 'org.springframework.boot' version '2.2.3.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testCompile 'com.jayway.jsonpath:json-path'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}
(base) [vagrant@workspace demo]$
(base) [vagrant@workspace demo]$ vim build.gradle
(base) [vagrant@workspace demo]$ cat build.gradle
plugins {
	id 'org.springframework.boot' version '2.2.3.RELEASE'
	id 'io.spring.dependency-management' version '1.0.8.RELEASE'
	id 'java'
    id "org.sonarqube" version "2.8"
}

group = 'com.example'
version = '0.0.1-SNAPSHOT'
sourceCompatibility = '1.8'

repositories {
	mavenCentral()
}

dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-web'
	testCompile 'com.jayway.jsonpath:json-path'
	testImplementation('org.springframework.boot:spring-boot-starter-test') {
		exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
}

test {
	useJUnitPlatform()
}
(base) [vagrant@workspace demo]$
~~~


### 配置 profile

在 **`~/.gradle/gradle.properties`** 里添加 

~~~
# gradle.properties
systemProp.sonar.host.url=http://http://10.0.0.13:8000/sonarqube
~~~

或者直接在命令行中添加


~~~
-Dsonar.host.url=http://http://10.0.0.13:8000/sonarqube  -Dsonar.verbose=true
~~~


## 进行代码扫描

进入 demo 项目执行扫描

~~~
(base) [vagrant@workspace demo]$ gradle sonarqube -Dsonar.host.url=http://10.0.0.13:8000/sonarqube  -Dsonar.verbose=true

> Task :sonarqube
SonarScanner will require Java 11 to run starting in SonarQube 8.x
SCM provider autodetection failed. Please use "sonar.scm.provider" to define SCM of your project, or disable the SCM Sensor in the project settings.
Classes not found during the analysis : [javax.annotation.meta.When]

Deprecated Gradle features were used in this build, making it incompatible with Gradle 7.0.
Use '--warning-mode all' to show the individual deprecation warnings.
See https://docs.gradle.org/6.1/userguide/command_line_interface.html#sec:command_line_warnings

BUILD SUCCESSFUL in 7s
5 actionable tasks: 1 executed, 4 up-to-date
(base) [vagrant@workspace demo]$
~~~

## 查看扫描结果

在 Project 中可以看到 demo 项目的汇总信息

![sonarqube](/assets/img/sonarqube/sonarqube34.png)

进入指定 demo 项目，查看详情

![sonarqube](/assets/img/sonarqube/sonarqube35.png)

到此 SonarQube 与 Gradle 的集成就演示完毕了

---

# 总结

一个敏捷开发团队一定会需要一套静态代码扫描软件

**[SonarQube][sonarqube]** 就是这样的软件

上面的步骤描述了 **[SonarQube][sonarqube]** 与 Gradle 的集成过程

后面接着讲如何与将这个过程集成到 Jenkins 的 pipeline 中

* TOC
{:toc}


---

[sonarqube]:https://www.sonarqube.org/
[sonarscanner_for_gradle]:https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-gradle/